# CDK Intrinsic Validator

This CDK construct allows you to add intrinsic validation to your CDK stacks.
Adding intrinsic validation adds checks that occur during deployment that, if
they fail, will automatically roll back the stack.

**Example error**
![An example of an intrinsic validation error](images/failure-example.png)

## Usage

```ts
import * as ecs from '@aws-cdk/aws-ecs';
import {
  IntrinsicValidator,
  FargateValidationFactory,
  Validation
} from '@wheatstalk/cdk-intrinsic-validator';

const cluster = new ecs.Cluster(...);

const curlImage = ecs.ContainerImage.fromRegistry('curlimages/curl:7.78.0');
// A convenience tool for creating Fargate validations that have some
// common options (i.e., a specific ecs cluster.)
const fargateValidations = new FargateValidationFactory(this, 'FargateValidationFactory', {
  cluster,
});

// Validate the stack on every deploy and fail the deployment if any of
// the given validations fail so that CloudFormation can auto-rollback.
new IntrinsicValidator(this, 'IntrinsicValidator', {
  validations: [
    // Always succeeds. Not necessary. But even it succeeds, if anything
    // else in this validations list fails, the intrinsic validator will
    // error and the stack deployment will roll back.
    Validation.alwaysSucceeds(),
    // Public endpoints because in this vpc environment, I have no
    // services.
    fargateValidations.runContainer(curlImage, 'https://www.example.com/'),
    fargateValidations.runContainer(curlImage, 'https://www.amazon.ca/'),
    fargateValidations.runContainer(curlImage, 'https://www.google.com/'),
    // Uncomment and this validation will fail and roll back the stack.
    // fargateValidations.runContainer(curlImage, 'https://fake.fake.fake/'),
  ],
});
```