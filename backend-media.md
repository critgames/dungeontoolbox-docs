### Media
This file acts as the integration guide for developers. It explains exactly how the media service works internally and how the frontend team should implement it.

```markdown
# Media Service Design & Integration Guide

## 🎨 Design Philosophy

The Media Service (`media.dungeontoolbox.com`) acts as a **Smart Reverse Proxy**. Instead of exposing Supabase Storage URLs directly to the client (which causes CORS headaches and exposes backend paths), we sit Nginx in the middle.

This allows us to:
1.  **Standardize Domains:** All requests go to `*.dungeontoolbox.com`.
2.  **Handle Auth Transparently:** Convert browser cookies into API tokens.
3.  **Control Caching:** Cache public assets aggressively, but never cache private user data.

## 🧩 Architecture Flow

1.  **Browser** makes a request to `media.dungeontoolbox.com`.
2.  **Nginx** checks the path:
    * If `/assets/...`: It forwards to the **Public** Supabase bucket.
    * If `/vault/...`: It checks for the `sb-access-token` cookie.
3.  **Nginx (Private Path Only):**
    * Extracts the cookie value.
    * Rewrites the request to add `Authorization: Bearer <token>`.
4.  **Supabase** validates the token against Row Level Security (RLS).
5.  **Response:** The image is returned to the browser with the correct CORS headers for `app.dungeontoolbox.com`.

## 💻 Frontend Client Setup

To ensure this works, the Frontend and API must be configured to share cookies correctly.

### 1. Cookie Configuration (Critical)
When the user logs in via the **API**, the Auth Cookie must be set with the following attributes. If the `Domain` is missing, the Media service cannot see the cookie.

```javascript
// Example: Setting the cookie in the API (Node/Express)
res.cookie('sb-access-token', token, {
  httpOnly: true,
  secure: true,          // Required for HTTPS
  sameSite: 'None',      // Required for Cross-Site requests
  domain: '.dungeontoolbox.com' // NOTICE THE LEADING DOT
});
```
