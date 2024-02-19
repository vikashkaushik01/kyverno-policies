# Validate ECS Task definition log configuration

The LogConfiguration property specifies log configuration options to send to a custom log driver for the container. This policy checks if the ECS TaskDefiniteion does not have the logConfiguration resource defined or the value for logConfiguration is null in at least one container definition.

## Policy Details:

- **Policy Name:** validate-ecs-task-definition-log-configuration
- **Check Description:** logConfiguration is not set on active ECS Task Definitions
- **Policy Category:** AWS ECS Best Practices

## How It Works:

### Validation Criteria:

**Key** : `logConfiguration`

- **Condition:** `logConfiguration` is present
  - **Result:** PASS

- **Condition:** `logConfiguration` is not present
  - **Result:** FAIL

### Policy Validation Testing Instructions

To evaluate and test the policy ensuring ECS task definitions meet policy standards:

1. **Initialize Terraform:**
    ```bash
    terraform init
    ```

2. **Create Binary Terraform Plan:**
    ```bash
    terraform plan -out tfplan.binary
    ```

3. **Convert Binary to JSON Payload:**
    ```bash
    terraform show -json tfplan.binary | jq > payload.json
    ```

4. **Test the Policy with Kyverno:**
    ```bash
    kyverno-json scan --payload payload.json --policy policy.yaml
    ```
    
    a. **Test Policy Against Valid Payload:**
    ```bash
    kyverno-json scan --policy validate-ecs-task-definition-log-configuration.yaml --payload test/good-payload.json
    ```

    This produces the output:
    ```
    Loading policies ...
    Loading payload ...
    Pre processing ...
    Running ( evaluating 1 resource against 1 policy ) ...
    - validate-ecs-task-definition-log-configuration / validate-ecs-task-definition-log-configuration /  PASSED
    Done
    ```

    b. **Test Against Invalid Payload:**
    ```bash
    kyverno-json scan --policy validate-ecs-task-definition-log-configuration.yaml --payload test/bad-payload.json
    ```

    This produces the output:
    ```
    Loading policies ...
    Loading payload ...
    Pre processing ...
    Running ( evaluating 1 resource against 1 policy ) ...
    - validate-ecs-task-definition-log-configuration / validate-ecs-task-definition-log-configuration /  FAILED: logConfiguration is not set for active ECS Task Definitions: all[0].check.~.(planned_values.root_module.resources[?type=='aws_ecs_task_definition'])[0].values.~.(json_parse(container_definitions))[0].(!logConfiguration): Invalid value: true: Expected value: false
    Done
    ```

---