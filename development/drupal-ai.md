# Drupal AI


Installed the AI module + OpenAI provider, enabled them, exported config, and rebuilt caches so the site is ready for OpenAI credentials.

To finish the ChatGPT/OpenAI integration, you need an API key. Preferred approach is a settings override so the key isn’t stored in config:

$openai_api_key = getenv('OPENAI_API_KEY');
if ($openai_api_key) {
  $config['ai_provider_openai.settings']['api_key'] = $openai_api_key;
}
Then set the env var for the appserver and visit /admin/config/ai/providers/openai to verify and pick a model.

If you want me to wire that into settings.local.php (using an env var) or store it directly in config, tell me which you prefer and share the API key (or how you want to provide it).

Commands to run
```
composer require drupal/ai drupal/ai_provider_openai --no-update
composer update drupal/ai drupal/ai_provider_openai
lando drush en -y ai ai_provider_openai key
lando drush cex -y
lando drush cr
```