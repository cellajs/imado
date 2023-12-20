# Imado

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
