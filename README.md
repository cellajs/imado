# Imado (+ TUS + AWS Lambda + S3)
Imado is built as a configure-once solution for file handling with the TUS protocol, for public as well as for private files. It supports images and files such as PDF and DOCX and runs on AWS Lambda. Data - including cache - is stored on your own S3 bucket. Optionally it also supports videos.

This repo is a helper to use Imado and is currently being used by [CellaJS](https://github.com/cellajs/cella). The required AWS lambda repo is currently private.

[Contact us here](https://cellajs.com/contact) if you are interested in using it or email us at <info@cellajs.com>.

## Server

Wrap tus + s3-store and the Imado API to start uploading to your api.

```typescript
import { ImadoTus } from "imado";
import { env } from "env"; // or wherever you store your env variables

const tus = ImadoTus({
  secret: env.TUS_UPLOAD_API_SECRET,
  credentials: {
    bucket: env.AWS_S3_UPLOAD_BUCKET,
    region: env.AWS_S3_UPLOAD_REGION,
    accessKeyId: env.AWS_S3_UPLOAD_ACCESS_KEY_ID,
    secretAccessKey: env.AWS_S3_UPLOAD_SECRET_ACCESS_KEY,
  },
});

tus.listen({
  host: "0.0.0.0",
  port: Number(env.TUS_PORT ?? "1080"),
});
```

## Signed (private) URLs

Generate signed URLs for your private CDN (if the url does not include the signUrl it will just pass through).

```typescript
import { ImadoUrl } from "imado";
import { env } from "env";

export const getImadoUrl = new ImadoUrl({
  signUrl: env.PRIVATE_CDN_ROOT,
  cloudfrontKeyId: env.AWS_CLOUDFRONT_KEY_ID,
  cloudfrontPrivateKey: env.AWS_CLOUDFRONT_PRIVATE_KEY,
});

const thumbnailUrl = `${env.PRIVATE_CDN_ROOT}/thumbnail.jpg`;

const signedThumbnailUrl = getImadoUrl.generate(thumbnailUrl, {
  width: 100,
  format: "avif",
});
```

## What to do on your application

### Backend

- Create an endpoint with a JWT including a `public` boolean and `sub` in the payload. Recommended would also to define a `expiresIn` (exp). For example:

```typescript
const jwt = require("jsonwebtoken");

async function getUploadToken(ctx) {
  const isPublic = ctx.req.query("public");
  const user = ctx.get("user");

  const token = jwt.sign(
    { sub: user.id, public: isPublic === "true" },
    process.env.TUS_UPLOAD_API_SECRET
  );

  return ctx.json({
    success: true,
    data: token,
  });
}
```

### Frontend

You would include something like [Uppy](https://uppy.io/) and uppy/tus.

```typescript
import Uppy from "@uppy/core";
import Tus from "@uppy/tus";

// Get the JWT from your server
const { token } = await (await fetch("/api/upload/token?public=true")).json();

// Convience method to read out JWT (base64) string
const readJwt = (token: string) => JSON.parse(atob(token.split(".")[1]));

// Read out JWT
const { public: isPublic, sub } = readJwt(token);

const uppy = new Uppy({
  meta: {
    public: isPublic,
  },
})
  .use(Tus, {
    endpoint: "http://localhost:1080", // endpoint for tus
    removeFingerprintOnSuccess: true,
    headers: {
      authorization: `Bearer ${token}`, // pass the JWT to tus
    },
  })
  .on("complete", (result) => {
    if (result.successful) {
      const urls = result.successful.map((file) => {
        const uploadKey = file.uploadURL.split("/").pop();
        return new URL(`https://path_to_your_cdn.com/${sub}/${uploadKey}`);
      });

      // Do something with the urls!
    }
  });
```
