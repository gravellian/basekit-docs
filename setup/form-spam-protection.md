# Public Forms and Spam Protection

BaseKit standardizes on Webform for public contact and submission forms. Drupal
core Contact is uninstalled by the BaseKit recipe helper so a Standard-profile
installation does not retain an unexpected second contact endpoint.

The BaseKit recipe installs and configures two no-key protections:

- Antibot protects every Webform submission form and any core Contact form left
  enabled by an existing site.
- Honeypot protects all forms by default, with login, search, exposed Views,
  and its own settings form excluded. It enables a five-second minimum
  completion time and logs rejected submissions.

These controls are the portable baseline, not a guarantee that all automated
spam will be stopped. A public site receiving browser-driven or human-assisted
spam should add a challenge such as Cloudflare Turnstile and configure it on
each anonymous submission form. Challenge credentials are site-specific and
must not be committed to BaseKit or a site's exported configuration.

Every new public form must be checked before launch:

1. Confirm its form ID and public route.
2. Confirm Antibot and Honeypot markup is present for an anonymous visitor.
3. Set a reasonable per-user/IP submission limit in Webform.
4. Add a site-specific challenge if passive controls do not stop observed spam.
5. Verify the intended notification and do not expose a second form system.

For existing sites, `lando recipes-apply` installs the baseline modules and
imports their shared configuration. Review configuration changes before
deployment because enabling protection globally can affect custom forms.
