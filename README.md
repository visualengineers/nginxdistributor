# Fresh Deployment

* Clone the repository, then:
* Rename `nginx/conf/app.conf` to `nginx/conf/app.ttmp`
* Rename `nginx/conf/app.tmp` to `nginx/conf/app.conf`
* Run `docker compose up` in root directory of repository
* You can now test that everything is working by running:

```bash
docker compose run --rm  certbot certonly --webroot --webroot-path /var/www/certbot/ --dry-run -d example.org`
```

* Replace `example.org` with your domain name! You should get a success message like "The dry run was successful".
* Re-run Certbot without the --dry-run flag to fill the folder with certificates

```bash
docker compose run --rm  certbot certonly --webroot --webroot-path /var/www/certbot/ -d example.org
```

* Now rename `nginx/conf/app.conf` to `nginx/conf/app.tmp` and
* Rename `nginx/conf/app.ttmp` to `nginx/conf/app.conf`
* Restart your container using `docker compose restart`

See also:

* https://mindsers.blog/en/post/https-using-nginx-certbot-docker/
* https://dev.to/isthatcentered/testing-an-angular-build-with-base-href-locally-44af
* https://gist.github.com/soheilhy/8b94347ff8336d971ad0

# Renewing the certificates

One small issue you can have with Certbot and Let's Encrypt is that the certificates last only 3 months. You will regularly need to renew the certificates you use if you don't want people to get blocked by an ugly and scary message on their browser.

But since we have this Docker environment in place, it is easier than ever to renew the Let's Encrypt certificates!

```bash
docker compose run --rm certbot renew
```

This small "renew" command is enough to let your system work as expected. You just have to run it once every three months. You could even automate this process…

# Important Commands

Start the environment

```bash
docker compose up
```

Update the Environment

```bash
docker compose restart
```

If you want to avoid service interruptions (even for a couple of seconds) reload it inside the container using:

```bash
docker compose exec webserver nginx -s reload
```

# Add a new Reverse Proxy Location

You can add new locations in `nginx/conf/sites/*.conf` by adding new files such as:

```nginx
location /leac/ {
    rewrite /leac/(.*) /$1 break;
    proxy_pass http://itv21.informatik.htw-dresden.de:4203;
    proxy_set_header Origin http://$host;
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
    proxy_http_version 1.1;
}
```

Simply replace the location name and the port in `proxy_pass` for the internal docker container. 

# Considerations for Angular Apps

Angular apps need to specify a `base-href` argument in the build for the deployment. Make sure to run the build command with this option:

```bash
ng build --configuration production --base-href /leac/
```

In the Angular App, you can either use correct relative URLs to the correct assets folders etc. or try to access the APP_BASE_HREF by adding an injector in `app.module.ts`:

```js
import { APP_BASE_HREF, PlatformLocation } from "@angular/common";

export function getBaseHref(platformLocation: PlatformLocation): string {
  return platformLocation.getBaseHrefFromDOM();
}
```

and

```js
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [
    {
      provide: APP_BASE_HREF,
      useFactory: getBaseHref,
      deps: [PlatformLocation]
    }
  ],
  bootstrap: [AppComponent]
})
```

You can then inject this information in a component with:

```js
export class AppComponent {
  title = 'ng-rootdir';
  
  constructor(@Inject(APP_BASE_HREF) public baseHref:string) {
  }
}
```

And access the value in your template like this:

```html
<div>
  <img [src]="baseHref + '/assets/img/comingsoon.jpg'" 
       alt="logo" 
       class="logo">
</div>
```


See also: 

* https://wildermuth.com/2021/08/08/Using-Angular-s-Base-HREF-in-Paths/
* https://stackoverflow.com/questions/39287444/angular2-how-to-get-app-base-href-programmatically#46493276