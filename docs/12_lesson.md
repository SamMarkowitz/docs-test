
#Lesson 12 - Existing Content

##Goal
In this lesson we'll learn how to integrate ready-made content into our flow.

##Get Started
Instead of printing that our flow has completed, let's send an email to HR to let them know that the new hire's email address has been created and the status of the new hire's equipment order. If you're using a pre-built CLI you'll have a **/content** folder that contains all of the ready-made content.

##Ready-Made Operation
We'll use the **send\_mail** operation which is found in the **/base/mail** folder. All ready-made content begins with a commented explanation of its purpose and its inputs, outputs and results.

Here's the documentation of **send\_mail**:

```yaml
####################################################
#   This operation sends a simple email.
#
#   Inputs:
#       - hostname - email host
#       - port - email port
#       - from - email sender
#       - to - email recipient
#       - cc - optional - Default: none
#       - bcc - optional - Default: none
#       - subject - email subject
#       - body - email text
#       - htmlEmail - optional - Default: true
#       - readReceipt - optional - Default: false
#       - attachments - optional - Default: none
#       - username - optional - Default: none
#       - password - optional - Default: none
#       - characterSet - optional - Default: UTF-8
#       - contentTransferEncoding - optional - Default: base64
#       - delimiter - optional - Default: none
#   Results:
#       - SUCCESS - succeeds if mail was sent successfully (returnCode is equal to 0)
#       - FAILURE - otherwise
####################################################

```

When calling the operation, we'll need to pass values for all the arguments listed in the documentation that are not optional.

You might notice that in general operation and flow inputs are named using snake_case. This is in keeping with Python conventions, especially when using an operation that has a `python_script` type action. The `send_mail` operation though, uses a `java_action` so its inputs follow the Java camelCase convention.

##Imports
First, we'll need to set up an import alias for the new operation since it doesn't reside where our other operations and subflow do.

```yaml
imports:
  base: tutorials.base
  hiring: tutorials.hiring
  mail: org.openscore.slang.base.mail
```

##Task
Then, all we really need to do is create a task in our flow that will call the `send_mail` operation. We need to pass a host, port, from, to, subject and body. You'll need to substitute the values in angle brackets (`<>`) to work for your email host. Notice that the body value is taken directly from the `print_finish` task with the slight change of turning the `\n` into a `<br>` since the `htmlEmail` input defaults to true.  

```yaml
    - send_mail:
        do:
          mail.send_mail:
            - hostname: "'<hostname>'"
            - port: <port>
            - from: <email from>
            - to: <email to>
            - subject: "'New Hire: ' + first_name + ' ' + last_name"
            - body: >
                'Created address: ' + address + ' for: ' + first_name + ' ' + last_name + '<br>' +
                'Missing items: ' + missing + ' Cost of ordered items:' + cost
```

##Run
We can run the flow now and see that the ordering takes place, the proper information is aggregated and then it is printed.

```bash
run --f <folder path>/tutorials/hiring/new_hire.sl --cp <folder path>/tutorials/base,<folder path>/tutorials/hiring,<content folder path>/org/openscore/slang/base --i first_name=john,last_name=doe
```

##Up Next
In the next lesson we'll see how to use system properties to send values to input variables.

##New Code - Complete

**new_hire.sl**
```yaml
namespace: tutorials.hiring

imports:
  base: tutorials.base
  hiring: tutorials.hiring
  mail: org.openscore.slang.base.mail

flow:
  name: new_hire

  inputs:
    - first_name
    - middle_name:
        required: false
    - last_name
    - missing:
        default: "''"
        overridable: false
    - total_cost:
        default: 0
        overridable: false

  workflow:
    - print_start:
        do:
          base.print:
            - text: "'Starting new hire process'"

    - create_email_address:
        loop:
          for: attempt in range(1,5)
          do:
            hiring.create_user_email:
              - first_name
              - middle_name:
                  required: false
              - last_name
              - attempt
          publish:
            - address
          break:
            - CREATED
            - FAILURE
        navigate:
          CREATED: get_equipment
          UNAVAILABLE: print_fail
          FAILURE: print_fail

    - get_equipment:
        loop:
          for: item, price in {'laptop': 1000, 'docking station':200, 'monitor': 500, 'phone': 100}.items()
          do:
            hiring.order:
              - item
              - price
          publish:
            - missing: fromInputs['missing'] + unavailable
            - total_cost: fromInputs['missing'] + cost
        navigate:
          AVAILABLE: print_finish
          UNAVAILABLE: print_finish

    - send_mail:
        do:
          mail.send_mail:
            - hostname: "'<host name>'"
            - port: <port>
            - from: <email from>
            - to: <email to>
            - subject: "'New Hire: ' + first_name + ' ' + last_name"
            - body: >
                'Created address: ' + address + ' for: ' + first_name + ' ' + last_name + '<br>' +
                'Missing items: ' + missing + ' Cost of ordered items:' + cost

    - print_finish:
        do:
          base.print:
            - text: >
                'Created address: ' + address + ' for: ' + first_name + ' ' + last_name + '\n' +
                'Missing items: ' + missing + ' Cost of ordered items:' + cost

    - on_failure:
        - print_fail:
            do:
              base.print:
                - text: "'Failed to create address for: ' + first_name + ' ' + last_name"
```


