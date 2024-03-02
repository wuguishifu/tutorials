# How to Deploy a Static Vite React App on AWS using S3, CloudFront, Route53, and Google Domains, and how to add a GitHub CI/CD action.

## Section 1: creating a production build of the react app
1. Navigate to the root folder of your app in a terminal or command prompt.
2. Run the command `npm run build` to build a production dist of your app.
    * this may be a different command if you're using a different tool to scaffold your app like create-react-app.
3. Locate the build files. Default in `./dist`. The dist folder should be structured as follows:
    ```
    App
    ├── dist
    │   ├── assets
    │   │   ├── index-###.css
    │   │   ├── index-###.js
    │   │   ...
    │   ├── favicon.ico
    │   ├── index.html
    │   ├── ...
    ...
    ```

## Section 2. deploying the app into a new S3 bucket
1. Navigate to the S3 console (s3.console.aws.amazon.com) and create a new bucket.
2. In the public access settings section, unselect "Block _all_ public access" and check the section labelled "I acknowledge that the current settings might result in this bucket and the objects within becoming public".
3. Navigate into the bucket in the S3 console.
4. Upload the dist files from Section 1 to the root of the bucket. Under objects, you should see `favicon.ico`, `index.html`, and the `assets` folder.
5. Navigate to the "Permissions" tab of the bucket.
6. Under bucket policy, click edit and add the following policy:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
            }
        ]
    }
    ```
7. Navigate to the "Properties" tab of the bucket.
8. At the bottom, enable "Static website hosting".
    1. Click "edit".
    2. Select "Enable" under "Static website hosting".
    3. Select "Host a static website" under "Hosting type".
    4. Under "Index document" and "Error document - _optional_" enter "index.html".
        * react apps using routing will require get requests to certain urls that do not have a S3 bucket endpoint. Because the app handles this internally, we need to make sure that S3 knows that it should forward requests that don't have an S3 bucket endpoint to the react app, or "index.html".
    5. Click "Save changes".

## Section 3. registering a domain with Google Domains.
1. If you already have a domain registered, skip to Section 4.
2. Navigate to Google Domains (domains.google.com) select "Get a new domain" on the far left nav menu.
    * Here, you can search for and purchase domains.
3. Once you've selected and registered a domain, continue to Section 4.

## Section 4. creating a SSL certification using AWS Certificate Manager
1. Navigate to the ACM console (us-east-1.console.aws.amazon.com/acm) and click "Request a certificate".
    * Ensure you are in N. Virginia (us-east-1) at the top right next to your account name.
2. Select "Request a public certificate" and click "Next".
3. Under "Fully qualified domain name", enter any domain and subdomains you need. For example, I'm using "bo-bramer.com", "www.bo-bramer.com", "freelance.bo-bramer.com", and "gated.bo-bramer.com".
4. Under "Validation method" selec "DNS validation - recommended".
5. Click "Request".
6. Under your list of certificates, click on the one for the domain name you just requested. Under status, it should say "Pending validation".
7. In the "Domains" section, note where the CNAME name and CNAME values are located for each of the subdomains you're attempting to validate.
8. In a new tab, navigate to the Default name servers section of your Google domain.
    1. Navigate to Google Domains.
    2. Under "My domains" select the domain you are attempting to validate.
    3. Click on "DNS" on the far left nav menu.
    4. Click on "Manage custom records"
9. Create new records for each row in the ACM Domains section (from Step 7)
10. Make each of these records Type CNAME with a TTL of 60. Under "Host name", input the "CNAME name" value from ACM. Make sure that you delete the extra ".your-domain.ext" part, as ACM automatically appends that part to the CNAME name. Under "Data", input the "CNAME value" from ACM.
11. In ACM, wait until each of the subdomains goes from "Pending validation" to "Success"
12. Once this happens, you may remove the CNAME custom records from your DNS settings in Google Domains.

## Section 5. creating a CloudFront distribution to serve your AWS S3 bucket
1. Navigate to the AWS CloudFront console (us-east-1.console.aws.amazon.com/cloudfront).
    * Ensure you are in N. Virginia (us-east-1) at the top right next to your account name.
2. Click on "Create distribution" on the top right
3. Under "Origin Domain", select your S3 bucket. It should give a warning telling you to use the S3 website endpoint rather than the bucket endpoint. Click on "Use website endpoint".
    * If this does not appear, you can navigate to your S3 bucket and manually copy your S3 website endpoint.
4. Scroll down and select your WAF preference.
5. Under "Settings>Alternate domain name (CNAME) - _optional_", click "Add item" and add the domain you registered earlier. In my case, I will be entering "bo-bramer.com".
6. Under "Custom SSL certificate - _optional_", click "Choose certificate" and select the ACM Certificate we just created in Section 4.
7. Click on "Create distribution".
8. Wait for the status to go from "Deploying" to "Enabled". In the meantime, continue on to Section 6. You do not need to keep this tab open.

## Section 6. adding a Route 53 hosted zone layer for your CloudFront distribution
1. Navigate to the AWS Route 53 console (us-east-1.console.aws.amazon.com/route53).
    * Ensure you are in N. Virginia (us-east-1) at the top right next to your account name.
2. Navigate to the "Hosted zones" tab using the far left nav menu.
3. Click on "Create hosted zone".
4. Under domain name, input the domain name you registered with earlier.
5. Under "Type", select "Public hosted zone".
6. Click "Create hosted zone".
7. Create a new record for your CloudFront distribution.
    1. Under the "Records" section of your hosted zone, click on "Create record".
    2. Input a subdomain if necessary.
    3. Under "Record type", select "CNAME".
    4. Toggle "Alias" to true
    5. Under "Route traffic to", select "Alias to CloudFront distribution".
    6. Click on the search bar and select your CloudFront distribution you made in Section 5.
    7. Click "Save"
8. In the "Records" section of your hosted zone, copy the "Value/Route traffic to" of the record of type NS. It should be 4 lines long and look something like this:
    ```
    ns-918.awsdns-48.net.
    ns-417.awsdns-23.com.
    ns-1515.awsdns-32.co.uk.
    ns-1101.awsdns-12.org.
    ```
9. Navigate to your domain in Google Domains.
10. In the "DNS" section of your domain, click on the "Custom name servers" tab.
11. In the "Name servers" section, add each line as a name server. Once you're finished, clicked "Save".
12. At the top of this page, click on "Switch to these settings".
13. Wait for the update to propagate, and your react app should be available at the domain you registered earlier.

## Section 7. setting up a AWS access credential
1. In AWS, go to the IAM console (us-east-1.console.aws.amazon.com/iamv2)
2. Scroll down to "Access keys" and create a new access key.
3. Make sure to copy the access key and the secret access key. We will need them in step 8.

## Section 8. setting up automated CI with GitHub Actions
1. Open up the GitHub repository associated with your project.
2. Go to the "Actions" tab and create a new workflow by clicking on the "New workflow" button.
3. Make a blank workflow by clicking "set up a workflow yourself".
4. Now, you can either directly in the GitHub editor, or you can commit the empty file and pull it to your local repo and edit it with your IDE. It will be located at ./github/workflows/main.yml
5. These files are in .yml format, and contain the name, when it runs, and what jobs to perform. For automatic building and deploying on S3, we're going to add the following sections to our code
    ```yml
    name: CI # this is the name of your action script

    on: # specifies that the script should run when updates are pushed to the master branch
      push:
        branches: [ master ]
    
    jobs:
      build:
        # runs-on and strategy specify what container environment to install
        runs-on: ubuntu-latest
        strategy:
          matrix:
            node-version: [18.16.0]
        
        # steps is a list of steps that the container should run. In here, we're going to specify what steps to take
        steps:
          # run on a Node.js platform
          - name: Use Node.js $${{ matrix.node-version }}
            uses: actions/setup-node@v1
            with:
              node-version: $${{ matrix.node-version }}
        
          # checkout the code from the master branch
          - name: Git checkout
            uses: actions/checkout@v2
        
          - name: Install packages
            run: |
              npm install
        
          - name: Build for production
            run: |
              npm run build
            
          # here we're going to use Jake Jarvis's easy deploy script. This handles uploading files, automatically assign mime types, etc
          - name: Deploy to S3
            uses: jakejarvis/s3-sync-action@master  
            with:
              args: --acl public-read --delete
            env:
              AWS_S3_BUCKET: $${{ secrets.AWS_BUCKET_NAME }}
              AWS_ACCESS_KEY_ID: $${{ secrets.AWS_ACCESS_KEY_ID }}
              AWS_SECRET_ACCESS_KEY: $${{ secrets.AWS_SECRET_ACCESS_KEY }}
              AWS_REGION: $${{ secrets.AWS_REGION }}
              SOURCE_DIR: "/dist" # whever your build folders are located
    ```
6. Now, we need to set our secrets in GitHub. In your GitHub repository, go to settings and locate the "Secrets and variables" tab.
7. Under "Actions", create your repository secrets.
    * the bucket name should be `AWS_BUCKET_NAME`
    * the access key id should be `AWS_ACCESS_KEY_ID`
    * the secret access key should be `AWS_SECRET_ACCESS_KEY`
    * the aws region should be `AWS_REGION`
8. Now, your GitHub actions script should be ready to go. Pushing any changes to the Master branch should start the script, and deploy to S3.

Congrats!
