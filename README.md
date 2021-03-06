# Terraform AWS Lambda module

## About:

Setup of a Lambda function with CloudWatch Alarms.

The following boolean variables control which alarms will be enabled;

- __create_errorRate_alarm:__  Alarms on errors with a default of threshold of 1 percent during a 5 minute measurement period
- __create_throttleCount_alarm:__ Alarm on throttle count of 1 within 1 minute measurement period
- __create_iteratorAge_alarm:__ Alarm for Stream based invocations such as Kinesis, alerts you when the time to execute is over 1 minute within a 5 minute measurement period
  - More on the iteratorAge metric [here](https://aws.amazon.com/premiumsupport/knowledge-center/lambda-iterator-age/)
- __create_deadLetterQueue_alarm:__ Alarm for DLQueue messages (for async Lambda invocations or SQS queues for example), 1 message within 1 minute triggers the alarm.

This AWS Lambda module can integrate with the following:
- __API Gateway:__ The parameter ```create_api_gateway_integration``` is set to ```true``` by default, supply the ```api_gateway_rest_api_id```
- __SNS Topic:__ The parameter ```create_sns_topic_subscription``` is set to ```false``` by default. 
- __SQS:__ The parameter ```create_sqs_integration``` is set to ```false``` by default. This can create an event source mapping to an SQS queue.

## How to use:

Use the code below in combination with the tf-iam module (```module.iam.role_arn```) to set the correct IAM policies.

```terraform
module "lambda" {
  source = "github.com/rpstreef/tf-lambda?ref=v1.0"

  namespace         = var.namespace
  region            = var.region
  resource_tag_name = var.resource_tag_name

  lambda_function_name = local.lambda_function_name
  lambda_role_arn      = module.iam.role_arn
  lambda_layer_arn     = var.lambda_layer_arn

  lambda_memory_size = var.lambda_memory_size
  lambda_timeout     = var.lambda_timeout

  lambda_environment_variables = {
    NAMESPACE = var.namespace
    REGION    = var.region

    COGNITO_USER_POOL_CLIENT_ID = var.cognito_user_pool_client_id
    COGNITO_USER_POOL_ID        = var.cognito_user_pool_id

    DEBUG_SAMPLE_RATE = var.debug_sample_rate
  }

  create_deadLetterQueue_alarm = false
  create_iteratorAge_alarm     = false

  api_gateway_rest_api_id = var.api_gateway_rest_api_id
}
```

## Changelog

### v1.3
 - Removed lambda_filename variable, use CI/CD instead to deploy Lambda code
 
### v1.2
 - Added SQS event source integration
 
### v1.1
 - Added SNS Topic subscription sub-module

### v1.0
 - Initial release
