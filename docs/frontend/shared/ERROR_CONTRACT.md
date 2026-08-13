# Error Contract

## 401 Unauthorized
Clear local session and redirect to `/login` or `/(auth)/login`.

## 404 Not Found
Render NotFound component. Do not retry.

## 500 Internal Server Error
Show generic error toast. Do not crash app.
