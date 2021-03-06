# lecli

lecli is a gem that provides a CLI to generate Let's Encrypt certificates. The name stands for **L**et's **E**ncrypt **CLI**.

lecli wraps the lower level [ACME protocol Client gem](https://github.com/unixcharles/acme-client) with the intention to create your custom [Certbot](https://certbot.eff.org/). This would make it easier for you to automate/script around it. In order to achieve this, lecli pairs well with cron jobs and the recommended [whenever gem](https://github.com/javan/whenever).

## Installation

```
$ gem install lecli
```

## Getting started

The CLI will use the Let's Encrypt staging endpoint unless explicitly passed the `--production` flag. All other configuration data is managed by a config file - `lecli.yml`. To help understand the available options you can run the following in your terminal and a sample YAML file will be generated for you:

```
$ lecli yaml
```

Now let's see what's inside...

### `lecli.yml`

```
---
domains:
- example.com
- test.net
- yetanotherwebsite.com
common_name: example.com
account_email: test@account.com
request_key: request.pem
certificate_key: certificate.pem
challenges_relative_path: challenges
success_callback_script: deploy.sh
```

Only required options in this file are **domains** (list of domains), **common_name** and **account_email**. All others can be deleted if you're OK with the defaults, all of which will be loaded for you *except* **success_callback_script**. If the callback script is not specified nothing will be executed after a successful certificate request.

### The flow

From the two available types of validation requests only HTTP (and not DNS) is supported [yet](#contributing). This means you'll need to serve a token (lecli will create them for you) accessible from each domain in the **list of domain addresses** requested.

The tokens will be written to the **challenges_relative_path** and need to be served behind each domain you are requesting, i.e. `example.com/.well-known/acme-challenge/#{token_filename}` needs to return the token created. If requesting multiple domains at once you will probably need some additional setup to route from each domain requested to where the tokens are persisted.

An example of a simple deployment is when working with a single domain and lecli is executed on the host machine. If working with an nginx server you can just point the challenges path to write the tokens on `/usr/share/nginx/html/.well-known/acme-challenge/`. This way the tokens will be served so that Let's Encrypt is able to reach them.

![alt text](https://github.com/fdoxyz/lecli/blob/master/lecli_diagram.png)

After Let's Encrypt is able to access both tokens on the list of domain addresses requested the certificates can be issued. The resulting certificate will be identified by the **email** and under the **common_name** provided. The certificates (`.pem` files) can be renamed with **request_key** and **certificate_key**.

Optionally you can specify a script with **success_callback_script** to be executed. This script will function as a "callback hook" and it will run after successfully exporting the domains' certificate.

Now that you've read about `lecli.yml` options available (keywords in **bold**). If you've made sure to: (1) Customize the options config file to create the desired certificate, and (2) made sure the **challenges_relative_path** path is available for a public internet request, then you're now ready to kick off the validation process by executing the following on your terminal:

```
lecli generate
```

### Making use of the result Certificates

A simple example `nginx.conf` excerpt to make use of the result certificates could be the following

```
server {
  listen 443 ssl;
  server_name example.com;

  ssl_certificate       /etc/nginx/ssl/request.pem;
  ssl_certificate_key   /etc/nginx/ssl/certificate.pem;

  ...
}
```

You can script a server restart if needed, or any other setup that you require to make use of the newly created certificates. Just make sure to point the **success_callback_script** path in your config file (and make the script 'executable') so the CLI can automatically execute it if the request result was successful.

If you pair the CLI with a cron-job (specially using the [whenever](https://github.com/javan/whenever) gem) you've essentially put together a Let's Encrypt bot and can now leverage scripting for more complex deployments. Your certificates will be renewed periodically. When using **whenever** you'll have lecli CLI in your crontab as easy as:

```
every :month, at: '4am' do
  command "lecli --production -f /path/to/config/file.yml"
end`
```

Be sure to run `lecli help` for more details.

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `bundle exec rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment. To install this gem onto your local machine, run `bundle exec rake install`.

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/fdoxyz/lecli.

Please include tests if new features are added and make sure rubocop styling guide is met.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
