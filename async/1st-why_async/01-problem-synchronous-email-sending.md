# Problem: Why Do We Need Asynchronous Architecture?

## 1. Background

Imagine we are building an application where users can sign up, reset their passwords, or receive notifications through email.

Whenever a user performs an action, our application may need to send an email using an external email service provider.

For example, when a user signs up, the application sends a welcome email.

## 2. The Problem

**Should the user have to wait until the email has been sent before receiving a response?**

In a simple implementation, the application processes the request like this:

1. The user submits the signup form.
2. The application validates the user's information and saves the user in the database.
3. The application sends a welcome email and waits for the email provider's response.
4. Only after the email operation finishes does the application return a response to the user.

For a single user, or when the number of requests is low, this approach might work well enough.

But what happens when our application has hundreds or thousands of users signing up or requesting emails at the same time?

### Challenges as the Number of Users Increases

- **High response time:** Every request may have to wait for the email provider to respond.
- **Server resource usage:** Application requests remain active while waiting for email operations to finish.
- **Request timeouts:** Slow email providers can cause requests to exceed client, proxy, or server timeouts.
- **Limited scalability:** As concurrent requests increase, application servers can become overloaded with work that could run independently.
- **External dependency:** If the email provider is slow or unavailable, user-facing operations may also become slow or fail if they depend on email completion.
- **Failure handling:** If sending an email fails, the application needs a way to retry it without requiring the user to submit the original request again.

For example, if 1,000 users sign up within a short period, the application may need to handle 1,000 email operations alongside the signup requests. If each request waits for its email operation, response times and server resource usage can increase significantly.

The actual impact depends on the email provider, concurrency, server capacity, and implementation.

## 3. Expected Behavior

We want the application to:

- Validate the signup request and save the user successfully.
- Return a response without waiting for the email to be sent.
- Process email tasks in the background.
- Handle multiple email requests efficiently.
- Retry failed email operations when appropriate.
- Avoid losing pending email tasks when a server crashes or restarts.

## 4. Design Question

**How can we send emails in the background without making users wait, while supporting increasing traffic and handling failures reliably?**

We will explore different architectural approaches, starting with a simple solution and gradually improving it as we discover its limitations.

## 5. Scope

This case study focuses on asynchronous email processing, background workers, task storage, multiple application servers, scalability, and failure handling.

We will begin without a message queue and introduce additional components only when the limitations of the current design justify them.