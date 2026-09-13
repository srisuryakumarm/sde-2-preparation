# JWT & Authentication/Authorization Primer: The Missing Piece Before Day 67

**Where this fits:** insert this before Day 67 (JWT Authentication at the Gateway). Day 67's theory block explains *what a JWT is used for*, but never explains what one actually contains, how signature verification works, or the basic distinction between authentication and authorization — despite the day's own project task requiring you to implement JWT issuance and validation from scratch.

**Why this exists:** Day 67 has you reject requests with an invalid `Bearer` token, but "Bearer token," "signature," and "claims" are never defined before that point. Without this, implementing JWT validation is copying a pattern rather than understanding what it protects against.

**What this primer deliberately does *not* cover:** OAuth 2.0 (Day 67 itself already draws the distinction — "custom JWT issuance, not OAuth 2.0" — and that's as deep as this plan needs to go), refresh-token rotation strategies, and token revocation approaches. These are real, deeper topics, but none of them are required for Day 67's specific deliverable: Gateway-level validation of a signed Bearer token.

---

## Part 1: Authentication vs. Authorization

Two words that get used almost interchangeably in casual conversation, but mean genuinely different things in a system design:

- **Authentication** — proving *who you are*. Logging in with a username and password is authentication.
- **Authorization** — given who you are, deciding *what you're allowed to do*. Whether an authenticated user can view another user's order is authorization.

A JWT is fundamentally an **authentication** mechanism — proof of identity — but it commonly carries **authorization** data alongside it (a `role` claim, for instance), which is why the two get bundled together in practice even though they're conceptually separate questions.

---

## Part 2: What a JWT Actually Is

A JWT (JSON Web Token) is a string made of **three parts, separated by dots**: `header.payload.signature`. Each part is base64url-encoded.

- **Header** — metadata: which algorithm was used to sign this token. Example: `{"alg":"HS256","typ":"JWT"}`.
- **Payload** — the actual claims: data about the user or session. Example: `{"sub":"user123","role":"ADMIN","exp":1735689600}`. `sub` (subject) identifies who this token is about; `exp` is an expiry timestamp.
- **Signature** — proof the token hasn't been tampered with since it was issued.

**Critical, easy-to-miss detail:** base64 is *encoding*, not *encryption*. Anyone can decode the header and payload and read them in plain text — a JWT is not a secret container. Never put sensitive data (passwords, secrets) in the payload. The signature protects *integrity* (has this been altered?), not *confidentiality* (can anyone read it?).

---

## Part 3: How Signing and Verification Actually Work

- **Symmetric signing (HMAC, e.g., `HS256`)** — one shared secret key both creates and verifies the signature. Simple, but every service that needs to verify a token must hold that same secret.
- **Asymmetric signing (RSA, e.g., `RS256`)** — a private key signs; a public key verifies. The public key can be distributed freely; only whoever holds the private key can create a valid signature. This is why a Gateway can validate tokens without being able to forge new ones itself, when RSA is used.

The verification step, either way: the receiving side recomputes the signature from the received `header.payload` using the key it has, and checks whether that matches the signature that came with the token. If anyone altered the payload after issuance (say, changing `"role":"USER"` to `"role":"ADMIN"`), the recomputed signature won't match anymore — and the token is rejected. This is exactly what "the Gateway validates the signature and expiry on every request" (Day 67) means mechanically.

---

## Part 4: The Actual Flow

1. A user logs in (username/password). The server verifies the credentials, then creates a JWT containing whatever claims matter (user ID, role, expiry), and signs it.
2. The client stores that token and sends it on every subsequent request in the `Authorization` header: `Authorization: Bearer <token>`.
3. **"Bearer"** means exactly what it sounds like: whoever holds ("bears") this token is treated as authenticated for this request — no additional per-request proof needed, unlike a server-side session lookup.
4. The server (or, in Day 67's design, the Gateway sitting in front of every module) checks the signature and the `exp` claim on every incoming request, before the request is allowed to reach anything downstream.

---

## Coding Exercise

Do this in plain Java — no library, no Spring — to see there's no magic underneath it:

```java
import java.util.Base64;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;

public class MinimalJwtDemo {
    private static final String SECRET = "my-demo-secret-key";

    public static void main(String[] args) throws Exception {
        String header = "{\"alg\":\"HS256\",\"typ\":\"JWT\"}";
        String payload = "{\"sub\":\"user123\",\"role\":\"ADMIN\"}";

        String encodedHeader = base64UrlEncode(header);
        String encodedPayload = base64UrlEncode(payload);
        String unsignedToken = encodedHeader + "." + encodedPayload;

        String signature = sign(unsignedToken, SECRET);
        String token = unsignedToken + "." + signature;

        System.out.println("Token: " + token);
        System.out.println("Valid?   " + verify(token, SECRET));

        // Now tamper with the payload, WITHOUT re-signing, and re-check
        String tamperedPayload = base64UrlEncode("{\"sub\":\"user123\",\"role\":\"SUPERADMIN\"}");
        String tamperedToken = encodedHeader + "." + tamperedPayload + "." + signature;
        System.out.println("Tampered valid? " + verify(tamperedToken, SECRET));
    }

    private static String base64UrlEncode(String s) {
        return Base64.getUrlEncoder().withoutPadding().encodeToString(s.getBytes(StandardCharsets.UTF_8));
    }

    private static String sign(String data, String secret) throws Exception {
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), "HmacSHA256"));
        return Base64.getUrlEncoder().withoutPadding().encodeToString(mac.doFinal(data.getBytes(StandardCharsets.UTF_8)));
    }

    private static boolean verify(String token, String secret) throws Exception {
        String[] parts = token.split("\\.");
        String unsigned = parts[0] + "." + parts[1];
        String expectedSignature = sign(unsigned, secret);
        return expectedSignature.equals(parts[2]);
    }
}
```

Run it. Confirm the first check prints `true`, and the tampered one prints `false` — because the attacker changed the payload but has no way to produce a matching signature without knowing `SECRET`. That gap between "readable" and "trustworthy" is the entire reason a JWT is signed in the first place.

**Definition of done:** you can explain, out loud, without notes: the difference between authentication and authorization, what a JWT's three parts are and which ones are readable vs. which one protects integrity, the difference between HMAC and RSA signing at a conceptual level, and what a `Bearer` token actually is.

---

## Quick Reference

| Term | One-line definition |
|---|---|
| Authentication | Proving who you are |
| Authorization | Deciding what an authenticated identity is allowed to do |
| JWT | A signed, three-part token (`header.payload.signature`) proving identity/claims |
| Claim | A piece of data in a JWT's payload (e.g., `sub`, `role`, `exp`) |
| HMAC (symmetric) | One shared secret key both signs and verifies |
| RSA (asymmetric) | A private key signs; a public key verifies |
| Bearer token | A token where simply holding it is treated as proof of authentication |

### Daily Deliverable
- [ ] Can state the difference between authentication and authorization without notes.
- [ ] Can explain a JWT's three parts and why base64 encoding isn't encryption.
- [ ] Can explain, at a conceptual level, how signature verification catches tampering.
- [ ] Coding exercise complete — tampered token correctly fails verification.
- [ ] Ready for Day 67 with the mechanism understood, not just the annotation pattern.
