# ADR - Email Notification

**Context**: Sends notifications to users via email notifications, keeping them informed about relevant events or updates related to the feature. These notifications should be sent asynchronously to ensure efficient processing and scalability.

**Problem:**
1. We need to design a robust and scalable email notification system that can handle all user notifications

**Considered Options:**
1. **SMTP:**  Use the standard SMTP protocol for sending emails, which is widely supported by most servers.
2. **Using [SQS](https://aws.amazon.com/sqs/), [SNS](https://aws.amazon.com/sns/), and [SES](https://aws.amazon.com/ses/):** Amazon Simple Queue Service (SQS) for message queuing, Amazon Simple Notification Service (SNS) for message routing, and Amazon Simple Email Service (SES) for sending email notifications.

**Discarded Options:** 
1. **SMTP:** SMTP, while being the standard protocol for email transmission, presents several challenges in the context of our requirements:
    - **Complexity in managing email delivery:** SMTP requires managing servers, configuring software, and ensuring reliable delivery, which can be complex.
    - **Scalability challenges:** SMTP setups may struggle to handle large volumes of emails efficiently, requiring additional infrastructure and management overhead to scale.
    - **Potential issues with deliverability and spam filtering:** SMTP setups might face challenges with email deliverability and getting flagged as spam, especially when sending a high volume of emails
    - Lack of built-in features for handling bounce management and improving deliverability: SMTP setups often lack features like dedicated IP addresses and built-in bounce handling capabilities, which are crucial for ensuring reliable email delivery.

**Decision:** We'll implement the email notification system using SQS, SNS, and SES. While both SMTP and SES are based on the SMTP protocol, SES offers several advantages tailored to our requirements.
1. SQS and SNS are fully managed services provided by AWS, offering automatic scaling to handle varying message volumes and traffic spikes. Also, eliminating the need for managing servers and infrastructure for email delivery.
2. SQS stores the notification message in the SQS queue and ensures reliable message delivery, while SNS simplifies message routing and fan-out to multiple subscribers.
3. SES automatically scales to handle varying message volumes and provides reliable delivery through features like bounce handling and reputation monitoring.
4. SES offers features such dedicated IP addresses to improve email deliverability by establishing a positive sending reputation. It also provides mechanisms for managing bounces and complaints, enhancing overall email deliverability.
5. Using AWS services like SQS, SNS, and SES provides cost-effectiveness through pay-as-you-go pricing.

**Consequences:**
1. Team members will need to familiarize themselves with AWS services such as SQS, SNS, and SES.

**References**
https://aws.amazon.com/ses/details/
https://docs.aws.amazon.com/ses/latest/dg/send-email-concepts-deliverability.html

