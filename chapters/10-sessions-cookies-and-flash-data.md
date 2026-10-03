# 10. Sessions, cookies, and flash data

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Validation and form errors](./09-validation-and-form-errors.md) | [Notes index](../README.md) | [Next: Filters, security, and file uploads](./11-filters-security-and-file-uploads.md) |

## Store small per-visitor state in a session

A session stores server-side state associated with a browser session. Use the session service for values that need to be available across requests, such as a short-lived authenticated user identifier or a small workflow state.

```php
$session = session();

$session->set('user_id', $user->id);
$userId = $session->get('user_id');

$session->remove('temporary_filter');
```

Do not store a full database record or large object graph in the session. Store a small identifier and load the current data through the normal application and authorization paths.

After successful authentication, rotate the session identifier before relying on the authenticated state:

```php
$session = session();
$session->regenerate(true);
$session->set('user_id', $user->id);
```

On logout, remove or destroy the session as part of the sign-out flow. If you call `destroy()`, do not perform more session operations during that request.

## Show a one-request flash message

Flash data is session data intended to be read on the next request. A redirect can attach one message:

```php
return redirect()
    ->to('/profile')
    ->with('message', 'Profile updated.');
```

Read and escape it in the destination view:

```php
<?php $message = session()->getFlashdata('message'); ?>

<?php if ($message !== null): ?>
    <p role="status"><?= esc($message) ?></p>
<?php endif ?>
```

Flash data is useful for confirmations and short-lived errors. Use a database record for information that must persist beyond the next request.

## Configure session storage

The session config controls the driver, save path, cookie settings, and expiry. Local writable-file sessions need a writable `writable/session` directory. A production app may use a database or another supported handler based on its deployment and availability requirements.

Do not set session expiry or storage behavior casually. Check the session guide, PHP settings, and hosting environment together, then verify that sign-in and sign-out behave correctly after deployment.

## Set cookies deliberately

Cookies are stored by the browser and sent with later requests. Use them only for values that need to live on the client. The cookie configuration should use HTTPS in production and choose `HttpOnly` and `SameSite` values that fit the use case.

```php
$this->response->setCookie(
    'display_mode',
    'dark',
    60 * 60 * 24 * 30,
    '',
    '/',
    '',
    true,
    true,
    'Lax',
);
```

This example stores a non-sensitive display preference for thirty days. The arguments enable Secure and HttpOnly and set SameSite to Lax. Use the Cookie configuration defaults where possible so attributes are consistent across the app.

Read a cookie from the request:

```php
$displayMode = $this->request->getCookie('display_mode');
```

Do not store passwords or raw authentication secrets in a cookie. If an app needs a remember-me feature, use a carefully designed, revocable token rather than treating a browser value as trusted identity. A cookie value is untrusted input and must not grant access by itself.

When setting a cookie and returning a redirect response, check the response guide's `withCookies()` behavior so the cookie is copied to the redirect response.

## Session and cookie checklist

- Store only small, necessary state in a session.
- Regenerate the session identifier after successful authentication.
- Keep one-time messages in flash data.
- Use Secure cookies on HTTPS deployments.
- Set HttpOnly unless client-side scripts truly need access.
- Choose SameSite behavior for the actual cross-site flow.
- Treat every cookie value as untrusted input.
- Verify the configured storage path and permissions in production.

## Practice

1. Store a small preference in the session and remove it.
2. Show a flash message after a successful form submission.
3. Confirm a flash message disappears after the next request.
4. Set and read a non-sensitive preference cookie.
5. Test session behavior over HTTPS in a production-like environment.

## References

- [Session library](https://codeigniter.com/user_guide/libraries/sessions.html)
- [Cookies](https://codeigniter.com/user_guide/libraries/cookies.html)
- [HTTP responses and cookies](https://codeigniter.com/user_guide/outgoing/response.html)
