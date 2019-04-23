title: Form Security
summary: Ensure Forms are secure against Cross-Site Request Forgery attacks, bots and other malicious intent.

# Form Security

Whenever you are accepting or asking users to input data to your application there comes an added responsibility that it
should be done as safely as possible. Below outlines the things to consider when building your forms.

## Cross-Site Request Forgery (CSRF)

SilverStripe protect users against [Cross-Site Request Forgery](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF) 
(known as `CSRF`) by adding a SecurityID [HiddenField](api:SilverStripe\Forms\HiddenField) to each [Form](api:SilverStripe\Forms\Form) instance. The `SecurityID` contains a 
random string generated by [SecurityToken](api:SilverStripe\Security\SecurityToken) to identify the particular user request vs a third-party forging fake 
requests.

<div class="info" markdown="1">
For more information on Cross-Site Request Forgery, consult the [OWASP](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF) 
website.
</div>

The `SecurityToken` automatically added looks something like:


```php
use SilverStripe\Forms\Form;

$form = new Form(..);
echo $form->getSecurityToken()->getValue();

// 'c443076989a7f24cf6b35fe1360be8683a753e2c'
```

This token value is passed through the rendered Form HTML as a [HiddenField](api:SilverStripe\Forms\HiddenField).

```html
<input type="hidden" name="SecurityID" value="c443076989a7f24cf6b35fe1360be8683a753e2c" class="hidden"  />
```

The token should be present whenever a operation has a side effect such as a `POST` operation.

It can be safely disabled for `GET` requests as long as it does not modify the database (i.e a search form does not 
normally require a security token).


```php
$form = new Form(..);
$form->disableSecurityToken();
```

<div class="alert" markdown="1">
Do not disable the SecurityID for forms that perform some modification to the users session. This will open your 
application up to `CSRF` security holes.
</div>

## Strict Form Submission

To reduce attack exposure forms are limited, by default, to the intended HTTP verb (mostly `GET` or `POST`). Without 
this check, forms that rely on `GET` can be submitted via `POST` or `PUT` or vice-versa potentially leading to 
application errors or edge cases. If you need to disable this setting follow the below example:


```php
$form = new Form(..);

$form->setFormMethod('POST');
$form->setStrictFormMethodCheck(false);

// or alternative short notation..
$form->setFormMethod('POST', false);
```

## Spam and Bot Attacks

SilverStripe has no built-in protection for detailing with bots, captcha or other spam protection methods. This 
functionality is available as an additional [Spam Protection](https://github.com/silverstripe/silverstripe-spamprotection) 
module if required. The module provides an consistent API for allowing third-party spam protection handlers such as 
[Recaptcha](http://www.google.com/recaptcha/intro/) and [Mollom](https://mollom.com/) to work within the `Form` API. 

## Data disclosure through HTTP Caching (since 4.2.0)

Forms, and particularly their responses, can contain sensitive or user-specific data. 
Forms can prepopulate submissions when a form is redisplayed with validation errors,
and they by default contain CSRF tokens unique to the user's session.
This data can inadvertently be stored either in a user's browser cache or in an intermediary
cache such as a CDN or other caching-proxy. If incorrect `Cache-Control` headers are used, private data may be cached and
accessible publicly through the CDN.  

To ensure this doesn't happen SilverStripe adds `Cache-Control: no-store, no-cache, must-revalidate` headers to any 
forms that have validators or security tokens (all of them by default) applied to them; this ensures that CDNs
(and browsers) will not cache these pages.
See [/developer_guides/performance/http_cache_headers](Performance: HTTP Cache Headers).

## Related Documentation

* [Security](../security)

## API Documentation

* [SecurityToken](api:SilverStripe\Security\SecurityToken)