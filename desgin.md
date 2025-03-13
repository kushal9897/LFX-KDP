# Proposal: Combined Design for Kyverno CLI apply and test Commands

## 1. Overall Purpose

### apply Command:
Simulates the application of mutation policies (MAP/MP) on Kubernetes resource manifests in a dry-run mode. This shows the final mutated resource (as YAML or JSON) without modifying any live resources.

### test Command:
Executes policies against predefined test cases to validate policy behavior. It compares the actual mutated resource with the expected output from a test manifest and generates a report showing pass/fail results.

## 2. Inputs and Outputs

### For the apply Command

#### Inputs:
- **Policy Files:**
  - YAML or JSON files defining mutation policies.
  - Input via positional arguments or `-p` flag.
- **Resource Files:**
  - YAML or JSON files representing Kubernetes resources.
  - Input via `--resource` (or `-r`) flag.
- **Optional Flags:**
  - `--output (-o)`: Specifies a file or directory where mutated outputs are saved.
  - `--namespace (-n)`: Limits resource scope to a specific namespace.
  - `--set`: Passes variable overrides to policies.
  - `--cluster`: Fetches resources directly from the cluster.
  - `--stdin`: Allows piping in resource definitions.
  - `--json`: Supports JSON payloads.
  - `--detailed-results`: When enabled, prints the full mutated YAML for each resource.

#### Outputs:
- **Mutated Resource YAML/JSON:**
  - Displayed on the console or written to a file if `--output` is specified.
- **Status and Log Messages:**
  - Indicates whether a policy was applied, skipped, or encountered errors.
- **Detailed Output:**
  - When the `--detailed-results` flag is active, prints the entire mutated resource in YAML.

### For the test Command

#### Inputs:
- **Test Manifest File:**
  - A YAML file (e.g., `kyverno-test.yaml`) that defines test cases.
  - Each test case includes:
    - **Policy Files:** Policies to test.
    - **Input Resources:** The resources to be mutated.
    - **Expected Outcome:** The expected mutated output or pass/fail expectation.
- **Additional Flags:**
  - `--resource-path`: Directory containing test resource files.
  - `--detailed-results`: Displays detailed diffs for failed tests.
  - `--test-case-selector`: Filters and runs a subset of tests.

#### Outputs:
- **Test Report:**
  - A summary table or list showing pass/fail status for each test case.
- **Detailed Diff (if enabled):**
  - Line-by-line comparison of expected vs. actual output for failed tests.
- **Status Messages:**
  - Overall test summary indicating the number of passed, failed, or skipped tests.
- **Exit Codes:**
  - Exit code `0` for success (all tests pass) or a non-zero value if any test fails.

## 3. File Locations and Code Changes

### For the apply Command

- **Primary File:**
  - `cmd/cli/kubectl-kyverno/apply/apply.go`

- **Key Areas to Modify:**

  1. **Input Parsing:**
     - Ensure proper handling of policy file paths and resource file paths.
     - Validate flags such as `--cluster`, `--output`, and `--detailed-results`.

  2. **Resource Loading and Policy Application:**
     - In the helper function (e.g., `applyCommandHelper`), load policies and resources.
     - Simulate policy application by processing mutation rules.

  3. **Output Generation:**
     - In the main processing loop, if the `--detailed-results` flag is enabled, print the complete mutated YAML (using a helper function like `common.MarshalYAML`).
     - Write the output to a file or directory if the `--output` flag is used.

  4. **Logging and Error Handling:**
     - Retain error and skipped policy handling.
     - Add clear messages indicating the input processed and the output generated.

### For the test Command

- **Primary File:**
  - `cmd/cli/kubectl-kyverno/test/test.go` (or a similar file managing test functionality)

- **Key Areas to Modify:**

  1. **Input Parsing:**
     - Parse the test manifest file (using a flag such as `-f` or a default file name).
     - Validate the manifest to ensure it includes required fields (policies, resources, expected outcomes).

  2. **Test Execution Logic:**
     - Load policies and resources as defined in the test manifest.
     - For each test case, simulate the policy application and compare the mutated resource to the expected output.

  3. **Output Generation:**
     - Generate a summary report for all test cases (table or list format).
     - If `--detailed-results` is enabled, include a detailed diff for any failed test cases.

  4. **Exit Handling:**
     - Use an exit code of `0` when all tests pass or a non-zero code if any test fails.

## 4. Workflow Summary (Combined)

### Input Phase:
- **For apply:**
  - User provides policy files, resource files, and optional flags (e.g., `--cluster`, `--output`, `--detailed-results`).
- **For test:**
  - User provides a test manifest file that specifies test cases (including input resources, policies, and expected outputs).

### Processing Phase:
- **Apply Command:**
  - Load and validate policies and resources.
  - Simulate the application of mutation rules.
  - If `--detailed-results` is enabled, output the full mutated resource YAML.
- **Test Command:**
  - Load the test manifest.
  - Execute each test case by applying policies and comparing results with expectations.
  - Collect and aggregate test results.

### Output Phase:
- **Apply Command:**
  - Print or save the mutated resource.
  - Display status messages (policy applied, skipped, error).
- **Test Command:**
  - Display a report showing pass/fail status for each test case.
  - Print detailed diffs for failures when `--detailed-results` is enabled.
  - Provide an overall summary of the test results.

## 5. Summary of Code Changes in Simple Terms

### For apply:
- Update flag parsing to support new flags (e.g., `--detailed-results` and `--output`).
- Enhance the main processing loop to check if `--detailed-results` is active, and if so, print the full mutated YAML.
- Maintain existing error and skip messages for consistency.

### For test:
- Add or update a command file to handle test manifests.
- Parse test cases from the manifest, run them by applying policies to resources, and compare results with expected outputs.
- Generate a comprehensive report (including detailed diffs if requested) and set appropriate exit codes.
