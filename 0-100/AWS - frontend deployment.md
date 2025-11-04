## What should be stored in a database?
- You can store data like user information
- You must never store objects like images, videos, webpages.
- Databases are not optimized to handle such objects
- Instead use Object Stores.

## AWS S3 - Storage
- S3 - Simple Storage Service is an object store.

## CDNs - Distribution
- Content Delivery Networks.
- They have servers located everywhere in the world. The file/content you want to share is distributed by the CDNs.
- Basically they cache the content and distribute to the requestors.
- There are multiple servers across the world called POPs(Points of Presence).
- They are used to distribute content which is fixed and can be cached. This is why backend cannot be distributed by CDNs.

## Deploying Frontend/Any Content over AWS
1. Create the build files. => convert to html/css/js in dist folder.
```
npm run build
```
2. Create a S3 bucket and upload all files there.
	1. Keep all the access blocked here. You can provide CDN limited access later.
3. Create a cloud front distribution. 
4. Provide access to the CDN by creating a OAC(Origin Access Control). This will require to update the S3 policy. They give a file that needs to be put in the bucket.
5. Set the default root object to index.html
6. Create a SSL certificate.
