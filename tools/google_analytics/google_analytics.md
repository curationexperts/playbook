# Installing Google Analytics in Hyrax

1. Create an environment variable on the server called `GA_TRACKING_CODE`.
2. Go to analytics.google.com's admin dashboard and select click on 'Tracking Info'
3. Copy the tracking ID. It will looks like: `UA-142424242-4`.
4. In the Hyrax initalizer,  add the env variable as the tracking code as the GA id around [line 39](https://github.com/curationexperts/laevigata/blob/99b10d67aa3cca452f5d444c1f3bfbd00464713b/config/initializers/hyrax.rb#L39).
5. Add the ID to the server and in the `-cm` repositories for the project.
6. After deploying the code go to the Tracking Code page in GA and click "Send test traffic" to test that the code is active.

# Installing Google Analytics in Blacklight

1. Create an environment variable on the server called `GA_TRACKING_CODE`.
2. Go to analytics.google.com's admin dashboard and select click on 'Tracking Info'
3. Copy the the global site tag. It will look like this:

```html
         <!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=UA-143063836-3"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'UA-5555555-5');
</script>

```

4. Add this markup to the `app/view/layouts/blacklight/base.html.erb` as the first tag in the `<head>` tag, but replace the tracking code with the env variable: `<%= ENV['GA_TRACKING_CODE']  %>`

```html
<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=<%= ENV['GA_TRACKING_CODE']  %>"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', '<%= ENV['GA_TRACKING_CODE']  %>');
</script>
```

5. Add the ID to the server and in the `-cm` repositories for the project.
6. After deploying the code go to the Tracking Code page in GA and click "Send test traffic" to test that the code is active.
