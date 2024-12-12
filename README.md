PoC Build and Deploy React into AWS S3
======================================



**1\. Pre-requests**
====================

*   AWS account
    
*   S3 bucket configured for static website hosting (follow instructions [here](https://medium.com/@dav.abdallah/build-your-static-website-using-aws-s3-and-cloudfront-a-step-by-step-guide-a6fd515d2096))
    
*   User has permission to deploy into S3 bucket with Access and secret Keys.
    
*   GitHub repository
    

**2\. Create a Basic React Application**
========================================

*   **Install Node.js and npm:** You can download and install them from [nodejs.org](http://nodejs.org/).
    
*   **Create a New React App:** Use Create React App to set up a basic React application.
    
```bash
npm config set legacy-peer-deps true   
```

```bash
npx create-react-app poc-portal-app
```

*   **Navigate to the Project Directory:**
    
```bash
cd poc-portal-app   `
```

**3\. Push the React Application to GitHub**
============================================

*   **Initialize Git**: If you haven't already, initialize a git repository in your project directory.
    
```bash
 git init 
```

*   **Add Remote Repository**: Add your GitHub repository as a remote.
    
```bash
git remote add origin https://github.com/your-username/your-repo.git
```

*   **Add and Commit Files**:
    

```bash
 git add .
```
```bash
git commit -m "Initial commit" 
```

*   **Add and Commit Files**:

```bash
git branch -M main  
```
```bash
git push -u origin main
```

4\. **Create GitHub Repository Secrets and Variables**
======================================================

**Create GitHub Repository Secrets** for AWS Keys, S3 bucket name and **variable** for AWS Region.

Navigate to your repository's settings and under "Secrets and variables" from the left sidebar click on actions

<img width="436" alt="image" src="https://github.com/user-attachments/assets/6e6ceea0-009f-43ce-9bae-13ff4e42284f" />


Add the below into **Repository secrets**

*   **Name**: AWS\_ACCESS\_KEY\_ID
    
    *   **Secret**: Write Access Key of the prerequisite user.
        
*   **Name**: AWS\_SECRET\_ACCESS\_KEY
    
    *   **Secret**: Write Secret Key of the prerequisite user.
        
*   **Name**: AWS\_S3\_BUCKET
    
    *   **Secret**: S3 Bucket name
        

Add the below into **Repository Variables**

*   **Name**: AWS\_REGION
    
    *   **Value:** Your region
        

5\. **Set Up a GitHub Actions Workflow**
========================================

1.  **Create a Workflow File**:Create a directory named **.github/workflows** in your GitHub repository, then create a file named **deploy.yml** inside it.
    
2.  **Define the Workflow**:In your **deploy.yml** file, add the configuration that will build your React app and deploy it to your S3 bucket.
    
```yml
name: Deploy React App to S3

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '18.8' 

      - name: Clean npm cache
        run: npm cache clean --force

      - name: Remove node_modules and package-lock.json
        run: |
          rm -rf node_modules
          rm -f package-lock.json

      - name: Install Dependencies
        run: |
          npm install --legacy-peer-deps
          npm install ajv@latest 
          npm install ajv-keywords@latest 
          npm ci

      - name: Build Project
        run: npm run build

      - name: Deploy to S3
        uses: jakejarvis/s3-sync-action@v0.5.1
        with:
          args: --delete
        env:
          AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: ${{ vars.AWS_REGION }}
          SOURCE_DIR: 'build'
          
      - name: Verify Deployment
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: ${{ vars.AWS_REGION }}
        run: aws s3 ls s3://${{ secrets.AWS_S3_BUCKET }}/
        
      - name: Display Website URL
        run: echo "Your website is hosted at http://${{ secrets.AWS_S3_BUCKET }}.s3-website-${{ vars.AWS_REGION }}.amazonaws.com"

```



**Here's a simplified explanation of the GitHub Actions workflow:**

This workflow automates the deployment of a React app to an S3 bucket whenever a push is made to the main branch.

**Key Steps:**

1.  **Checkout Code:** Fetches the latest code from the repository.
    
2.  **Set Up Node.js:** Sets up the Node.js environment for building the app.
    
3.  **Clean Up:** Removes old node\_modules and package-lock.json files to ensure a clean installation.
    
4.  **Install Dependencies:** Installs necessary packages for the project.
    
5.  **Build Project:** Builds the React app, creating the final files for deployment.
    
6.  **Deploy to S3:** Uploads the built files to an S3 bucket using the s3-sync-action.
    
7.  **Verify Deployment:** Lists the contents of the S3 bucket to confirm successful deployment.
    
8.  **Display Website URL:** Prints the URL of the deployed website.

   
    

6\. **Test the GitHub Actions Workflow**
========================================

1.  **Make a Change and Push**:Make a small change to your React app (e.g., change some text in **src/App.js**) and push it to the main branch. This will trigger the GitHub Actions workflow.
    
2.  **Check the Workflow**:Go to the Actions tab in your GitHub repository to see the workflow run. Ensure it completes successfully.
    
3.  **Verify Deployment**:After the workflow completes, check your S3 bucket.You should see the **build** folder contents deployed there.
