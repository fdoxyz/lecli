# lecli

lecli is a gem that provides a CLI to generate Let's Encrypt certificates. It wraps around the [ACME protocol Client gem](https://github.com/unixcharles/acme-client). It pairs well with cron jobs and the [whenever gem](https://github.com/javan/whenever) for a tighter grip on automation/scripting customization.

## Installation

    $ gem install lecli

## Getting started

The CLI will use the Let's Encrypt staging endpoint unless explicitly passed with the `--production` flag. All other configuration data is managed by a config file - `.lecli.yml`. To help understand the available options you can run the following in your terminal and a sample YAML file will be generated for you

```
$ lecli yaml
```

Now let's see what's inside

### `lecli.yml`

```
---
port: 80
domains:
- example.com
common_name: Let's Encrypt
account_email: test@account.com
request_key: request.pem
certificate_key: certificate.pem
challenges_relative_path: challenges
success_callback_script: deploy.sh
```

Most entries are optional, except those that specify the domains you are requesting and "identity fields". Meaning that at least **domains** (list of entries), **common_name** and **account_email** should always appear in order to perform a valid request.

### The flow

From the two available types of validation requests only HTTP (and not DNS) is supported [yet](#Contributing). This means you'll need to serve a token (lecli will create them) behind each domain in the **list of domain addresses** requested, on a certain **port**.

The tokens are written to a single **challenges_relative_path** and need to be served behind each domain you are requesting, i.e. `example.com/.well-known/acme-challenge/#{token_filename}`. If requesting multiple domains at once you will need additional setup to route from each domain requested to where the tokens are persisted. When working with a single domain, for example, you can just make this relative path write the tokens on `/usr/share/nginx/html/.well-known/acme-challenge/` if working with an nginx server.

![alt text](https://github.com/fdoxyz/lecli/blob/master/lecli_diagram.png)

After Let's Encrypt is able to access both tokens on the list of domain addresses requested the certificates can be issued. The resulting certificate will be identified by the **email** and under the **common_name** provided. The name of the `.pem` files can be customized with **request_key** and **certificate_key**.

Optionally you can specify a script filename with **success_callback_script**. This script will function as a "callback hook" and it will be run after successfully exporting the domains' certificate.

In this section you've read about all `lecli.yml` options available (in **bold**). Now, if you've made sure you: (1) Customized the options in this file to create the desired certificate, and (2) made sure the **success_callback_script** path is available for a public internet. You are now ready to kick off the validation process by executing the following on your terminal

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

You can script a server restart if needed, or any other setup that you require to make use of the newly created certificates. Just make sure to point the **success_callback_script** path in your config file so the CLI can automatically execute it if the request result was success.

If you pair the CLI with a cron-job (specially using the [whenever](https://github.com/javan/whenever) gem) you've essentially "hooked up" the same Let's Encrypt bot and now can leverage ruby scripting, which at least to me is generally a breeze. You can now generate your certificates periodically and customize whatever part of the process you need, for example using whenever:

```
every :month, at: '4am' do
  command "/path/to/config/file/lecli --production"
end
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `bundle exec rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment. To install this gem onto your local machine, run `bundle exec rake install`.

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/fdoxyz/lecli.

Please include tests if new features are added and make sure rubocop styling guide is met.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
