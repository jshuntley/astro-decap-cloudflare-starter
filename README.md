# Astro + Decap CMS + Cloudflare Template

A starter template for building a static site with [Astro](https://astro.build) and managing its content with [Decap CMS](https://decapcms.org/), all deployed on [Cloudflare Pages](https://pages.cloudflare.com/). This setup includes a Cloudflare Worker acting as a custom GitHub OAuth proxy, allowing you to authenticate via GitHub without relying on Netlify Identity or Git Gateway. Additionally, you can optionally use [Cloudflare Zero Trust (Access)](https://www.cloudflare.com/products/zero-trust/access/) to add an extra layer of security to your `/admin` page.

## Features

- **Astro** for building fast, content-focused static sites.
- **Decap CMS** for an editor-friendly content management UI.
- **Cloudflare Pages** for fast, globally-distributed static hosting.
- **Cloudflare Worker (OAuth Proxy)** for handling GitHub OAuth without external services.
- **GitHub Backend** for storing and versioning content as Markdown files directly in your repo.
- **(Optional) Cloudflare Zero Trust** for restricting access to the `/admin` interface to authorized users only.

## Prerequisites

- **Node.js and npm** installed locally.
- A **GitHub account** with a repository set up for your site’s content.
- A **Cloudflare account** with a domain configured (optional but recommended for custom domains).
- A **GitHub OAuth application** for authentication.

## Getting Started

You can either use this repository as a boilerplate to get started quickly or follow the [manual setup](#manual-setup) instructions to configure each component individually.

### Quick Start with Boilerplate

1. **Clone the Repository**

   ```bash
   git clone https://github.com/jshuntley/astro-decap-cloudflare-starter.git
   cd astro-decap-cloudflare-starter
   ```

2. **Install Dependencies**

   ```bash
   npm i
   ```

3. **Configure GitHub OAuth Proxy**

   - Follow [step 4](#4-configure-the-decap-proxy-worker) in the manual setup section below to set up and deploy the Cloudflare Worker.

4. **Set Up Cloudflare Zero Trust**

   - Follow [step 5](#5-cloudflare-zero-trust-setup) in the manual setup section below to secure your `/admin` page.

5. **Deploy to Cloudflare Pages**

   - Follow [step 6](#6-cloudflare-pages-setup) in the manual setup section below to deploy your site.

6. **Access the CMS**
   - Navigate to `https://your-domain.com/admin` and log in with GitHub to start managing your content.

### Manual Setup

Follow these steps to manually set up each component of the stack.

#### 1. Create an Astro Project

##### a. Initialize the Project

1. **Install Node.js and npm**
   Ensure you have [Node.js](https://nodejs.org/) installed on your machine. Verify installation:

   ```bash
   node -v
   npm -v
   ```

2. **Create a New Astro Project**

   ```bash
   npm create astro@latest my-astro-site
   cd my-astro-site
   npm install
   ```

3. **Build the Project as Desired**

   Customize your Astro project by adding pages, components, styles, and other assets.

   - **Example:** Create a simple homepage by editing `src/pages/index.astro`:
     ```astro
     ---
     title: "Home"
     ---
     <html>
       <head>
         <title>{title}</title>
       </head>
       <body>
         <h1>Welcome to My Astro Site</h1>
       </body>
     </html>
     ```

4. **Test the Project Locally**
   ```bash
   npm run dev
   ```
   Open your browser and navigate to `http://localhost:4321` to view your site.

#### 2. Include Decap CMS in the Project

##### a. Integrate Decap CMS

1. **Add Decap CMS Configuration**

   - **Create the Admin Directory:**

     ```bash
     mkdir -p public/admin
     ```

   - **Create `config.yml`:**
     In `public/admin/`, create a file named `config.yml` with the following content. Replace placeholders with your actual GitHub repository details.

     ```yaml
     backend:
       name: github
       repo: "your-username/your-repo" # Replace with your GitHub username/repo
       branch: main
       base_url: "https://decap.your-domain.com" # This subdomainis for the OAuth proxy
       auth_endpoint: "auth" # Default path for authentication

     site_url: "https://your-domain.com"

     media_folder: "public/images/uploads"
     public_folder: "/images/uploads"

     collections:
       - name: "pages"
         label: "Pages"
         folder: "src/pages"
         create: true
         slug: "{{slug}}"
         fields:
           - { label: "Title", name: "title", widget: "string" }
           - { label: "Body", name: "body", widget: "markdown" }
     ```

2. **Add Decap CMS Interface**

   - **Create `index.html`:**

     In `public/admin/`, create an `index.html` file with the following content:

     ```html
     <!DOCTYPE html>
     <html>
       <head>
         <meta charset="utf-8" />
         <meta
           name="viewport"
           content="width=device-width, initial-scale=1.0"
         />
         <title>Content Manager</title>
       </head>
       <body>
         <!-- Include the Decap CMS script -->
         <script src="https://unpkg.com/decap-cms@latest/dist/decap-cms.js"></script>
       </body>
     </html>
     ```

3. **Verify Decap CMS Integration**

   - **Run the Development Server:**
     ```bash
     npm run dev
     ```
   - **Access the CMS Interface:**
     Navigate to `http://localhost:4321/admin` in your browser. You should see the Decap CMS interface with the "Login with GitHub" button.

#### 3. GitHub OAuth Setup

##### a. Create a GitHub OAuth Application

1. **Navigate to GitHub Developer Settings**

   - Go to [GitHub Developer Settings](https://github.com/settings/developers).

2. **Register a New OAuth App**

   - Click on **"New OAuth App"**.

   - **Fill in the Application Details:**
     - **Application name:** `Decap CMS OAuth Proxy` (or any name you prefer).
     - **Homepage URL:** `https://your-domain.com` (your main site).
     - **Authorization callback URL:** `https://decap.your-domain.com/callback` (the subdomain specifically for the OAuth proxy).

3. **Save the OAuth App**
   - Click **"Register application"**.
   - **Note the Client ID and Client Secret:**
     You can save these to `decap-proxy/.env` file for local development.
     You'll need these credentials later when configuring the Cloudflare Worker.

#### 4. Configure the decap-proxy worker

- This is intended to operate on a subdomain of your domain. eg `decap.your-domain.com`

##### a. Clone and Set Up the Repository

1. **Clone the decap-proxy Repository**

   ```bash
   git clone https://github.com/sterlingwes/decap-proxy.git
   cd decap-proxy
   cp wrangler.toml.sample wrangler.toml
   ```

2. **Configure `decap-proxy/wrangler.toml`**

   - **Edit `decap-proxy/wrangler.toml`:**  
     Open `decap-proxy/wrangler.toml` in your preferred editor and make the following changes:

     ```toml
     name = "decap-proxy"
     main = "src/index.ts"
     compatibility_date = "2024-04-19"  # Update as needed
     compatibility_flags = ["nodejs_compat"]

     workers_dev = false
     route = { pattern = "decap.your-domain.com", zone_name = "your-domain.com" } # Replace with your domain
     ```

     - **Explanation:**
       - `workers_dev = false`: Disables the default `workers.dev` subdomain.
       - `route`:
         - `pattern`: The subdomain where the OAuth proxy will be accessible (e.g., `decap.your-domain.com`).
         - `zone_name`: Your main domain managed in Cloudflare (e.g., `your-domain.com`).

   - **Ensure the Worker Requests the Correct Scopes:**  
     If your repository is private, you need to request the `repo` scope. This allows Decap CMS to access and modify private repositories.

     In your `decap-proxy/src/index.ts` file, line 30,you can see the `scope` parameter is set to `'public_repo,user'`. This is the default scope for public repositories.

     ```typescript
     const authorizationUri = oauth2.authorizeURL({
       redirect_uri: `https://${url.hostname}/callback?provider=github`,
       scope: "public_repo,user", // change this to 'repo' if your repository is private
       state: randomBytes(4).toString("hex"),
     });
     ```

##### b. Deploy the Worker

While still in the `decap-proxy` directory, deploy the worker to Cloudflare.

1. **Login to Cloudflare via Wrangler**

   ```bash
   npx wrangler login
   ```

   - This command will open a browser window prompting you to authenticate with Cloudflare.

2. **Deploy the Worker**
   ```bash
   npm run deploy
   ```

##### c. Add Environment Variables to the Worker in Cloudflare

1. **Navigate to the Cloudflare Dashboard**

   - Go to [Cloudflare Dashboard](https://dash.cloudflare.com/).

2. **Access Workers Settings**

   - Click on **"Workers & Pages"** in the sidebar.
   - Select **"Workers"** and then choose your deployed worker (`decap-proxy`).

3. **Add Environment Variables**

   - Go to **"Settings"** > **"Variables"**.
   - Under **"Environment Variables"**, add the following **Secret Variables**:
     - **`GITHUB_OAUTH_ID`**: Your GitHub OAuth application's Client ID.
     - **`GITHUB_OAUTH_SECRET`**: Your GitHub OAuth application's Client Secret.

4. **Save the Variables**
   - Ensure both variables are saved correctly as secrets.

#### 5. Cloudflare Zero Trust Setup

##### a. Secure the `/admin` Page

1. **Navigate to Cloudflare Zero Trust Dashboard**

   - Go to [Cloudflare Zero Trust](https://dash.cloudflare.com/) dashboard.

2. **Set Up an Access Policy**

   - **Go to "Access" > "Applications":**

     - Click on **"Add an application"**.

   - **Choose "Self-hosted":**

     - Select **"Self-hosted"** as the application type.

   - **Configure Application Settings:**

     - **Application Name:** `Decap CMS` (or any name you prefer).
     - **Application Domain:** `https://your-domain.com/admin/*`
       - This ensures that any access to the `/admin` path is protected.
     - **Session Duration:** Choose as per your preference (e.g., 1 hour).

   - **Set Up Access Policies:**

     - **Policy Name:** `Allow Only Team Members`
     - **Action:** `Allow`
     - **Include:**
       - **Email Domains:** e.g., `@yourcompany.com` to restrict access to specific email domains.
       - **Individuals or Groups:** Specify individual emails or Cloudflare Access groups.
     - **Exclude:** (Optional) Define any exclusions if needed.

   - **Save the Application**
     - Review the settings and click **"Save"**.

#### 6. Cloudflare Pages Setup

##### a. Deploy Your Astro Site to Cloudflare Pages

1. **Push Your Astro Project to GitHub**

   - Initialize a Git repository if you haven't already:
     ```bash
     git init
     git add .
     git commit -m "Initial commit"
     git branch -M main
     git remote add origin https://github.com/your-username/your-repo.git  # Replace with your repo URL
     git push -u origin main
     ```

2. **Connect to Cloudflare Pages**

   - **Navigate to Cloudflare Dashboard:**

     - Go to [Cloudflare Dashboard](https://dash.cloudflare.com/).

   - **Access Pages:**

     - Click on **"Pages"** in the sidebar.

   - **Create a New Project:**
     - Click **"Create a project"**.
     - **Connect to Git Provider:**  
       Select GitHub and authorize Cloudflare to access your repositories.
     - **Select Repository:**  
       Choose your `your-username/your-repo` repository.

3. **Configure Build Settings**

   - **Framework Preset:**  
     Select **Astro** if available. If not, choose **None**.

   - **Build Command:**

     ```bash
     npm run build
     ```

   - **Build Output Directory:**
     ```bash
     dist
     ```
     - Astro outputs built files to the `dist` directory by default.

4. **Set Environment Variables (If Needed)**

   - If your Astro project requires environment variables (e.g., API keys), set them in the Cloudflare Pages project settings under **"Environment Variables"**.

5. **Deploy the Project**

   - Click **"Save and Deploy"**.
   - **Monitor the Build Process:**
     - Cloudflare Pages will automatically build and deploy your site. Monitor the deployment logs for any issues.

6. **Configure Custom Domain (Optional but Recommended)**

   - **Add a Custom Domain:**
     - After deployment, navigate to your Pages project settings.
     - Click **"Add Custom Domain"** and follow the prompts to link `https://your-domain.com` to your Cloudflare Pages site.
   - **Update DNS Records:**
     - Ensure that your domain's DNS settings in Cloudflare point to the correct Cloudflare Pages endpoints.

7. **Verify Deployment**
   - Visit `https://your-domain.com` to ensure your site is live.
   - Navigate to `https://your-domain.com/admin` to access the Decap CMS interface (now secured by Cloudflare Zero Trust).

#### 7. Final Adjustments and Testing

##### a. Update `config.yml` with OAuth Proxy Details

1. **Open `public/admin/config.yml`**

   - Ensure the `backend` section includes the correct `base_url` and `auth_endpoint`:

     ```yaml
     backend:
       name: github
       repo: "your-username/your-repo" # Your GitHub username/repo
       branch: main
       base_url: "https://decap.your-domain.com" # Your OAuth proxy domain
       auth_endpoint: "auth" # Path to the auth endpoint

     site_url: "https://your-domain.com"

     media_folder: "public/images/uploads"
     public_folder: "/images/uploads"

     collections:
       - name: "pages"
         label: "Pages"
         folder: "src/pages"
         create: true
         slug: "{{slug}}"
         fields:
           - { label: "Title", name: "title", widget: "string" }
           - { label: "Body", name: "body", widget: "markdown" }
     ```

##### b. Ensure OAuth Proxy Requests Correct Scopes

1. **Verify OAuth Proxy Scope**

   - Since your repository is private, ensure that the OAuth flow requests the `repo` scope. This is handled in your `oauth.ts` by setting the `scope` parameter to `'repo'`.

   - **In `decap-proxy/src/index.ts`:**
     ```typescript
     const authorizationUri = oauth2.authorizeURL({
       redirect_uri: `https://${url.hostname}/callback?provider=github`,
       scope: "repo", // Ensure 'repo' scope for private repos
       state: randomBytes(4).toString("hex"),
     });
     ```

2. **Redeploy the Worker After Changes**
   - If you made changes to the worker code, deploy the worker again:
     ```bash
     npm run deploy
     ```

##### c. Test the Complete Authentication Flow

1. **Access the CMS Interface**

   - Navigate to `https://your-domain.com/admin` in your browser.
   - **Cloudflare Zero Trust:**
     - You should be prompted by Cloudflare Access to authenticate (e.g., via SSO, email, etc.).
     - Only authorized users can proceed.

2. **Initiate GitHub Login**

   - Click on **"Login with GitHub"**.
   - **OAuth Proxy Flow:**
     - A popup should appear directing you to `https://decap.your-domain.com/auth`.
     - You’ll be redirected to GitHub’s OAuth authorization page.

3. **Authorize the Application**

   - Log in to GitHub (if not already logged in).
   - Authorize the OAuth application to access your repository with the required scopes.

4. **Complete the Authentication**

   - After authorization, you should be redirected back to `https://decap.your-domain.com/callback` where the OAuth proxy exchanges the code for an access token and communicates it back to Decap CMS.
   - The popup should close automatically, and you should now have access to the CMS interface.

5. **Verify CMS Functionality**
   - Try creating or editing a page to ensure that Decap CMS can commit changes to your GitHub repository.
   - Check your GitHub repo to confirm that the changes reflect correctly.

## Troubleshooting

If you encounter any issues during setup, consider the following troubleshooting steps:

### a. "Repo Not Found" Error

- **Possible Causes:**

  - Incorrect `repo` field in `public/admin/config.yml`.
  - OAuth token lacks necessary permissions. Line 30 in `decap-proxy/src/index.ts` is not set to `'repo'` for private repositories.

- **Solutions:**

  1. **Verify Repository Details:**

     - Ensure the `repo` field follows the format `username/repo-name`.
     - Double-check spelling and case sensitivity.

  2. **Check OAuth Scopes:**

     - Ensure the OAuth flow requests the `repo` scope for private repositories.
     - Confirm that the OAuth token has the `repo` permission.

  3. **User Permissions:**
     - Ensure the GitHub user authenticating has access to the private repository.
     - If the repo is within an organization, ensure the OAuth app has access to the organization's repositories.

### b. "Client ID Undefined" Error

- **Cause:**  
  The GitHub OAuth Client ID is not correctly set in the environment variables.

- **Solution:**

  1. **Verify Environment Variable Names:**

     - Ensure that in Cloudflare Workers settings, the variables are named exactly as referenced in the worker code (`GITHUB_OAUTH_ID` and `GITHUB_OAUTH_SECRET`).

  2. **Redeploy the Worker:**
     - After setting or correcting environment variables, redeploy the worker to apply changes:
       ```bash
       npm run deploy
       ```

### c. Authentication Flow Issues

- **Symptoms:**

  - Popup does not close automatically.
  - Unable to access CMS after login.

- **Solutions:**

  1. **Check OAuth Callback Logic:**

     - Ensure the worker's callback script correctly sends the token back to Decap CMS and closes the popup.

  2. **Browser Console Errors:**

     - Open developer tools in your browser and check for any JavaScript errors or failed network requests during the authentication process.

  3. **Popup Blockers:**
     - Ensure that your browser is not blocking popups from your site.

### d. Cloudflare Zero Trust Issues

- **Symptoms:**

  - Unable to access `/admin` even after authentication.

- **Solutions:**

  1. **Review Access Policies:**

     - Ensure that your Zero Trust policies correctly include the users who should have access.

  2. **Check DNS Settings:**

     - Ensure that `decap.your-domain.com` is correctly pointed in your DNS settings and that the worker is deployed properly.

  3. **Test Without Zero Trust:**
     - Temporarily disable Zero Trust to determine if it’s the source of the issue. If access works without it, review your Zero Trust configuration.

## License

This template is provided **as-is**, with no warranties. Please refer to the individual licenses of [Astro](https://github.com/withastro/astro/blob/main/LICENSE), [Decap CMS](https://github.com/decaporg/decap-cms/blob/main/LICENSE), and [Cloudflare](https://www.cloudflare.com/) for their respective terms and conditions.

---

## Additional Resources

- **Astro Documentation:** [https://docs.astro.build/](https://docs.astro.build/)
- **Decap CMS Documentation:** [https://decapcms.org/docs/](https://decapcms.org/docs/)
- **decap-proxy Documentation:** [https://github.com/sterlingwes/decap-proxy](https://github.com/sterlingwes/decap-proxy)
- **Cloudflare Workers Documentation:** [https://developers.cloudflare.com/workers/](https://developers.cloudflare.com/workers/)
- **Cloudflare Zero Trust Documentation:** [https://developers.cloudflare.com/cloudflare-one/](https://developers.cloudflare.com/cloudflare-one/)
- **GitHub OAuth Documentation:** [https://docs.github.com/en/developers/apps/building-oauth-apps/authorizing-oauth-apps](https://docs.github.com/en/developers/apps/building-oauth-apps/authorizing-oauth-apps)

Feel free to customize this template further to fit your project's specific needs. If you encounter any challenges or have questions about specific steps, don't hesitate to reach out for assistance!
