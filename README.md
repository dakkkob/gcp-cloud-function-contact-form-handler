# Contact Form Backend Service README

## Overview

This Cloud Function provides a backend service for handling submissions from contact forms on static websites, primarily focusing on validating and forwarding received data via email. It is designed to work with Google Cloud Platform (GCP) and utilizes Flask for creating web responses.

Key features include:
- **Email Validation:** Ensures that the sender and recipient email addresses are valid.
- **reCAPTCHA Validation:** Integrates with Google's reCAPTCHA to prevent spam submissions.
- **CORS Support (origin domain filtering):** Configures Cross-Origin Resource Sharing (CORS) to allow requests from specific origins/domains only.
- **Customizable Email Content:** Dynamically constructs the email body based on form data and domain-specific configurations.
- **Secure Email Sending:** Uses SMTP over SSL for secure email transmission.
- **Python3.12**
- **Reply-To:** Field is populated from the email inputted by user.
- **Email Subject:** is constructed from SMTP_SUBJECT field plus domain name of the originating request
- **Validation:** it is possible to bypass re-captcha validation for given validation domain, for purpose of having a functional ping

## Configuration

The function relies on a JSON configuration stored in an environment variable named `CONFIG`, which can also be a GCP Secret exposed as environment variable (that's a feature of GCP 2nd Generation Cloud Function. 
Configuration should contain:
- SMTP server details (host, port, username, and password) for sending emails.
- A default SMTP subject and sender details.
- Domain-specific configurations, including recipient email addresses, reCAPTCHA secret keys, and custom fields for the email body.
- Custom fields are listed as key-value mapping of "html form input name" - "field to be used in email body"

Example `CONFIG` structure:

```json
{
  "SMTP_SUBJECT": "Contact Form Submission",
  "SMTP_FROM_NAME": "Your Website Name",
  "SMTP_FROM_EMAIL": "no-reply@example.com",
  "SMTP_HOST": "smtp.example.com",
  "SMTP_PORT": 465,
  "SMTP_USERNAME": "your-username",
  "SMTP_PASSWORD": "your-password",
  "REQUIRED_FIELDS": "name,email,message,g-recaptcha-response",
  "SUCCESS_MESSAGE": "Thank you for your message. We will be in touch shortly.",
  "VALIDATION_DOMAIN": "",
  "DOMAIN_CONFIG":  {
	"example.com": {
		"recipient": "contact@example.com",
		"recaptcha_secret_key": "your-recaptcha-secret-key",
		"email_body_fields": {
          "name": "Name",
          "email": "Email",
          "message": "Message"
            }
		},
	"example2.com": {
		"recipient": "contact@example.com",
		"recaptcha_secret_key": "your-recaptcha-secret-key",
		"email_body_fields": {
          "name": "Name",
          "email": "Email",
          "message": "Message"
            }
		}	
	}
}
```

## Deployment Guide

This guide outlines the steps to deploy the Contact Form Backend Service using Google Cloud Platform (GCP). Before starting, ensure you have an SMTP account for sending emails and have registered your site with Google reCAPTCHA to get the necessary secret and site keys.

### Prerequisites

1. **SMTP Account:** You need access to an SMTP server with valid credentials (username and password). This server will be used to send emails from the contact form.

2. **Google reCAPTCHA Keys:** Register your website with Google reCAPTCHA to obtain a secret key and a site key. The secret key will be used to validate reCAPTCHA responses server-side, and the site key will be used in your website's frontend.

### Step-by-Step Deployment

#### 1. Create a GCP Secret

1. **Log in** to your Google Cloud Platform console.
2. **Navigate** to the **Security** > **Secret Manager** section.
3. **Create a new secret** with the JSON configuration for your application. This JSON should include your SMTP credentials, reCAPTCHA secret key, and any other configuration specified in the `CONFIG` structure outlined in the README.
4. **Name your secret** appropriately (e.g., `contact-form-config`) and **save** it.

#### 2. Deploy the Cloud Function

1. **Prepare your function code** by ensuring it is in a directory with any dependencies specified in a `requirements.txt` file.
2. **Open** the Cloud Functions section in the GCP console.
3. **Create a new function** and choose **2nd Generation** as the generation option.
4. **Configure the function:**
   - Set the **trigger type** to HTTP.
   - Set the **runtime** to Python 3.12.
   - Under the **Runtime, build, and connections settings**, click in **Security and image repo**, then **add secret reference**, select the secret and in **Reference method** select **Exposed as environment variable**, named `CONFIG`.
5. **Deploy** the function.

#### 3. Configure Your Website's Contact Form

1. **Update your contact form's frontend** to include the Google reCAPTCHA site key in the form.
2. **Configure the form to POST** to the URL provided by the deployed Cloud Function. This URL is displayed in the Cloud Function's details page in the GCP console after deployment.

#### 4. Test Your Setup

1. **Submit a test entry** through your contact form.
2. **Check the email** address configured as the recipient to ensure the message is received.
3. **Verify reCAPTCHA validation** and email sending functionalities are working as expected.

#### OPTIONAL: Functional ping

To ensure your function operates smoothly for both regular use and automated testing/monitoring, you can bypass the reCAPTCHA requirement for test requests. This involves setting the request's Origin header to match a predefined "VALIDATION_DOMAIN" in your configuration. For these test requests, use a "secret_key" within the domain's configuration block instead of the usual "recaptcha_secret_key". This approach allows you to validate the function's responsiveness without the need for reCAPTCHA verification. Here's an example configuration for the special testing domain:

 json ```
   "VALIDATION_DOMAIN": "validation.example.com",
   "DOMAIN_CONFIG":  {
	"validation.example.com": {
		"recipient": "validator@example.com",
		"secret_key": "your-secret-key",
		"email_body_fields": {
          "name": "Name",
          "email": "Email",
          "message": "Message"
            }
		}	
	}
```

The request will look as usual, even the g-recaptcha-response field is kept and used to provide the secret key for validation:

json ```
{
    "name": "John Doe",
    "email": "johndoe@example.com",
    "message": "This is a test message.",
    "g-recaptcha-response": "your-secret-key"
}
```

### Troubleshooting

If you encounter issues during deployment or operation, check Cloud Function logs in the GCP console for any errors or warnings that may indicate what went wrong.
