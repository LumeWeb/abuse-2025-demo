
**Building and Running a Portal for Abuse Reporting**

**I. Introduction**

This guide explains how to set up and run a portal with abuse reporting capabilities. It is intended for developers, system administrators, and anyone interested in deploying a content abuse reporting system. The portal consists of a core component, a dashboard, an admin panel, and an abuse reporting plugin. Please note that the admin webapp is currently in MVP status, but the API is reliable. The current repository includes copies of the Admin and Abuse API Swagger specifications.

**II. Using the Demo**

1.  **Accessing the Web Applications**

    *   **Important Note:** The webapps currently have hardcoded domains. The API is the most reliable component at this time. This issue will be resolved in the near future.

    *   For demonstration purposes, the following URLs (using the `abuse-demo.lumeweb.com` domain) can be used:

        *   Admin Panel: `https://admin.abuse-demo.lumeweb.com/login`
        *   Dashboard: `https://account.abuse-demo.lumeweb.com/login`
        *   Abuse Reporting App: `https://abuse.abuse-demo.lumeweb.com/`

    *   Default login credentials for the admin panel: `contact@lumeweb.com` / `1234567890`

2.  **Admin Panel:**

    * https://admin.abuse-demo.lumeweb.com/abuse
    * https://admin.abuse-demo.lumeweb.com/abuse/cases
    *   Key features to focus on in the admin panel:

        *   **Viewing and managing abuse cases:** Browse abuse reports.
        *   **Changing the status and urgency of cases:**  Update the status (e.g., Open, In Progress, Resolved) and urgency level (e.g., Low, Medium, High) of reports.
        *   **Blocking and unblocking reported content:** Block content associated with a report and reverse that action.

    *   The admin panel is currently in MVP status.

3.  **Abuse Reporting App:**

    *   File a report using a CID or URL (e.g., IPFS URL). An example is https://dweb.link/ipfs/QmdmQXB2mzChmMeKY47C43LxUdg1NDJ5MWcKMKxDu7RgQm.

4.  **Email Reporting:**

    *   Report abuse by sending an email to `abuse-demo@lumeweb.com`. The email is polled every 5 minutes.

**III. Sample Abuse Reports**

The repository includes several sample abuse reports that demonstrate different types of abuse cases:

* `illegal.txt` - Report of illegal or harmful content
* `malware.txt` - Report of malware distribution
* `phishing.txt` - Report of phishing campaign  
* `spam.txt` - Report of spam activity

These can be used as templates or test cases when submitting reports via email. The user side reporting API does not run the classification code since the user is able to select the type.

**IV. API Access**

*   All authentication is JWT Bearer (Authorization: Bearer) based.

*   Access the API documentation using the Swagger UI:

    *   `https://admin.abuse-demo.lumeweb.com/swagger`
    *   `https://admin.abuse-demo.lumeweb.com/swagger.json`
    *   `https://admin.abuse-demo.lumeweb.com/swagger.yaml`

**IV. Installation and Configuration**

1.  **Install xportal:**

    ```bash
    go install go.lumeweb.com/xportal/cmd/xportal@v0.2.14
    ```

    Verify the installation:

    ```bash
    xportal version
    ```

2.  **Build the Portal:**

    It is crucial to use the specified commit hashes for plugin compatibility.

    ```bash
    PORTAL_VERSION="0289ce0e249695e2bd878fa6c6360f2a840ffea0"  xportal build --with go.lumeweb.com/portal-plugin-dashboard@02612754124addccc78175944a2073f2a2e18f8b --with go.lumeweb.com/portal-plugin-admin@a07d48e38bbebfde72e84d978aff2efd80c80221 --with  go.lumeweb.com/portal-plugin-core@bd6f68e52c26de34e5bb3068b8bbf97a777e5f3f --with go.lumeweb.com/portal-plugin-abuse@49d4ee700beeaed0abe041844480c8481cf5dddc
    ```

    This command builds the `portal` executable.

3.  **Configure the Core Portal:**

    *   Create the directory:

        ```bash
        mkdir -p /etc/lumeweb/portal/
        ```

    *   Create the `core.yaml` file: `/etc/lumeweb/portal/core.yaml`

    *   **Starter Config Template:**

        ```yaml
        account:
            deletion_grace_period: 48
        cron:
            enabled: true
            queue_limit: 50
        db:
            charset: utf8mb4
            file: portal.db
            type: sqlite
        domain: X
        identity: X
        log:
            level: info
        mail:
            auth_type: plain
            from: X
            host: X
            password: X
            port: 465
            ssl: true
            username: X
        port: 8080
        portal_name: X
        post_upload_limit: 104857600
        storage:
            s3:
                access_key: X
                buffer_bucket: X
                endpoint: https://X
                region: us-east-1
                secret_key: X
            sia:
                key: X
                url: http://X
            tus:
                locker_mode: db
        ```

    *   **Configuration Options Explained:**

        *   `account`:
            *   `deletion_grace_period`: The number of hours an account can remain inactive before being deleted.
        *   `cron`:
            *   `enabled`: Enables or disables the cron scheduler.
            *   `queue_limit`: The maximum number of tasks that can be queued for the cron scheduler.
        *   `db`:
            *   `charset`: Character set for the database (e.g., `utf8mb4`).
            *   `file`: Path to the SQLite database file.
            *   `type`: Database type (e.g., `sqlite`).
        *   `domain`: The base domain for the portal. **Important:** The webapps currently have hardcoded domains. The API is the most reliable component at this time. This issue will be resolved in the near future.
        *   `identity`: A unique identity for the portal: a 12-word BIP39 seed.
        *   `log`:
            *   `level`: Logging level (e.g., `info`, `debug`, `warn`, `error`).
        *   `mail`: Configuration for sending emails (e.g., for account verification, notifications). Ensure secure email settings.
            *   `auth_type`: Authentication type for the email server (e.g., `plain`).
            *   `from`: The "from" address for emails sent by the portal.
            *   `host`: The hostname of the email server.
            *   `password`: The password for the email account.
            *   `port`: The port number for the email server.
            *   `ssl`: Whether to use SSL/TLS for the email connection.
            *   `username`: The username for the email account.
        *   `port`: The port the portal will listen on (default: 8080).
        *   `portal_name`: The name of the portal.
        *   `post_upload_limit`: Maximum size for uploaded content (in bytes).
        *   `storage`: Configuration for content storage (S3 or Sia).
            *   `s3`:
                *   `access_key`: S3 access key.
                *   `buffer_bucket`: S3 bucket for buffering uploads.
                *   `endpoint`: S3 endpoint URL.
                *   `region`: S3 region.
                *   `secret_key`: S3 secret key.
            *   `sia`:
                *   `key`: Sia API key.
                *   `url`: Sia API URL.
            *   `tus`:
                *   `locker_mode`: Locking mode for TUS uploads (e.g., `db`).

4.  **Configure the Abuse Plugin:**

    *   Create the directory:

        ```bash
        mkdir -p /etc/lumeweb/portal/plugins.d/abuse/service.d/
        ```

    *   Create the `abuse.email.yaml` file: `/etc/lumeweb/portal/plugins.d/abuse/service.d/abuse.email.yaml`

    *   **Starter Config Template:**

        ```yaml
        imap_host: X
        imap_mailbox: INBOX
        imap_password: X
        imap_port: 993
        imap_user: X
        poll_interval: 300
        receive_enabled: true
        ```

    *   **Configuration Options Explained:**

        *   `imap_host`: IMAP server hostname.
        *   `imap_mailbox`: IMAP mailbox to monitor (usually `INBOX`).
        *   `imap_password`: IMAP password.
        *   `imap_port`: IMAP port (usually 993 for SSL).
        *   `imap_user`: IMAP username.
        *   `poll_interval`: How often to check the mailbox (in seconds).
        *   `receive_enabled`: Enable/disable email reception.

**V. SSL Configuration (using Caddy):**

*   Using SSL is highly recommended.

*   Basic Caddyfile example:

    ```caddyfile
    yourdomain.com {
        reverse_proxy localhost:8080
    }
    ```

*   Caddy can automatically obtain TLS certificates. Refer to the Caddy documentation for details. This is a simplified example; more complex configurations may be needed for production environments.

**VI. Running the Portal**

1.  **Start the Portal:**

    *   Navigate to the directory containing the `xportal` executable.
    *   Run the command:

        ```bash
        ./portal
        ```

    *   To run the portal as a background service, consider using systemd or a similar process manager.

2.  **Verify the Portal is Running:**

    *   Check the logs for any errors.
    *   Access the portal in a web browser (e.g., `https://yourdomain.com`).

**VII. Conclusion**

This guide has outlined the steps to set up and run the portal. Experiment with the portal and provide feedback.
