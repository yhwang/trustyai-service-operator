# LM-Eval

LM-Eval is a service for large language model evaluation underpinned by two open-source projects: [lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness)
and [Unitxt](https://www.unitxt.ai). LM-Eval is integrated into the TrustyAI Kubernetes Operator. In this tutorial, you will learn:
- How to enable and set up the LM-Eval
- How to create an LMEvalJob CR to kick off an evaluation job and get the results

## Enable LM-Eval
To use the LM-Eval service, enable the LM-Eval controller on the trustyai-service-operator by the following steps:
- Check the `trustyai-service-operator-controller-manager` deployment on the project/namespace where you deploy the trustyai kubernetes operator.
- Use the following command to see the current arguments for the operator:
  ```sh
  oc get deploy trustyai-service-operator-controller-manager -o yaml | yq '.spec.template.spec.containers[0].args'
  ```
  Here is an example output:
  ```
  - --leader-elect
  - --enable-services
  - TAS
  ```
  In this case, only the trustyai-service is enabled. Currently, trustyai kubernetes operator supports two services: TrustyAI service(TAS) and LM-Eval service(LMES).
  To enable LM-Eval, run the command to update the deployment:
  `oc edit deploy trustyai-service-operator-controller-manager` and change the `TAS` to `TAS,LMES` which enables both TrustyAI and LM-Eval services.

## Settings for LM-Eval
There are some configurable settings for LM-Eval service and they are stored in the `trustyai-service-operator-config` configmap. Here is a list of properties for
LM-Eval
- lmes-detect-device (true|false): Detect if there is availble GPUs or not and assign the proper value for `--device` argument for lm-evaluation-harness.
  If GPU(s) is found, it uses the `cuda` as the value for `--device`, otherwise, it uses the `cpu` as the value.
- lmes-pod-image: The image for the LM-Eval job. The image contains the necessary Python packages for lm-evaluation-harness and Unitxt.
  The default value is "quay.io/trustyai/ta-lmes-job:latest".
- lmes-driver-image: The image for the LM-Eval driver. Check `cmd/lmes_driver` directory for detailed information about the driver.
  The default value is "quay.io/trustyai/ta-lmes-driver:latest"
- lmes-grpc-port: The internal port for the gRPC service. The default value is "8082"
- lmes-grpc-service: The internal service name of the gRPC service: The default value is "trustyai-service-operator-lmes-grpc".
- lmes-grpc-server-secret: The secret name for the gRPC server if you'd like to enable TLS on the internal gPRC service. 
- lmes-grpc-client-secret: The secret name for the gPRC client if you'd like to enable mTLS on the internal gPRC service.
- lmes-image-pull-policy: The image-pulling policy when running the evaluation job. The default value is "Always".
- lmes-default-batch-size (int): The default batch size when invoking the mode inference API. This only works for local models
  The default value is "8"
- lmes-max-batch-size (int): The max. batch size that users can specify in an evaluation job. The default value is "24".
- lmes-pod-checking-interval: The interval to check the job pod for an evaluation job. The default value is "10s".
- driver-report-interval: The interval that the driver reports job status. The default value is "10s".
