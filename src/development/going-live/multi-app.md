# Serving multiple applications

Platform.sh's route replacement logic works well for basic cases with a single domain.  For more complex cases involving multiple domains or multiple subdomains it is often easier to specify a production domain explicitly in `routes.yaml`.  The following sections show a few common cases.

## Multiple domains, separate applications

Scenario: `foo.com` and `bar.com` will both be served by a single project, but by different application containers.  Both also have `www.` variants that should redirect to the non-www versions.

Solution:

In this case it's easiest to ignore the `{default}` replacement entirely in production and rely on it only for development.  Your `routes.yaml` file should look something like this:

```yaml
"http://foo.{default}/":
    type: upstream
    upstream: "foo:http"

"http://bar.{default}/":
    type: upstream
    upstream: "bar:http"

"http://bar.org/":
    type: upstream
    upstream: "bar:http"

"http://foo.com/":
    type: upstream
    upstream: "foo:http"

"http://www.bar.org/":
    type: redirect
    to: "http://bar.org"

"http://www.foo.com/":
    type: redirect
    to: "http://understandingdrupal.com"
```

Then, register both `foo.com` and `bar.com` as domains in the Platform.sh UI.  You can provide an SSL certificate for each separately.

When deploying to master, you should then see output something like:

```text
Environment routes:
        http://bar.bar.com/ is served by application `bar`
        http://bar.com/ is served by application `bar`
        http://foo.bar.com/ is served by application `foo`
        http://foo.com/ is served by application `foo`
        http://www.bar.com/ redirects to http://bar.com
        http://www.foo.com/ redirects to http://foo.com
        https://bar.bar.com/ is served by application `bar`
        https://bar.com/ is served by application `bar`
        https://foo.bar.com/ is served by application `foo`
        https://foo.com/ is served by application `foo`
        https://www.bar.com/ redirects to http://bar.com
        https://www.foo.com/ redirects to http://foo.com
```

The various permutations of `foo.bar.com` and `bar.bar.com` can be ignored.  They're a side effect of the `{default}` entry, which is only useful for development.  The important entries are:

```text
        http://bar.com/ is served by application `bar`
        http://foo.com/ is served by application `foo`
        http://www.bar.com/ redirects to http://bar.com
        http://www.foo.com/ redirects to http://foo.com
        https://bar.com/ is served by application `bar`
        https://foo.com/ is served by application `foo`
        https://www.bar.com/ redirects to http://bar.com
        https://www.foo.com/ redirects to http://foo.com
```

Which is exactly what you want: `bar.com` and `foo.com` are served by their respective applications and the `www.` prefix versions will redirect to them.  Because the file did not specify any HTTPS routes Platform generated those for us automatically.  (Had it specified all HTTPS routes instead, Platform.sh would have automatically generated HTTP routes that redirect to HTTPS.)

Now, `bar---master-7rqtwti-3lrgvrfivd7rm.us.platform.sh` is the internal-canonical name of the `bar.com` site, and `foo---master-7rqtwti-3lrgvrfivd7rm.us.platform.sh` is the internal name for the `foo.com` site.  At your domain registrar, add a CNAME for each domain to its corresponding domain name on Platform.sh.

