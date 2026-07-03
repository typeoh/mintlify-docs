# Source: https://rasa.com/docs/reference/testing/coverage

On this page

To generate a coverage report, use the `--coverage-report` option with the `rasa test e2e` command. You can also specify an output directory for the report with the `--coverage-output-path` argument. By default, the coverage results are printed to `stdout` and saved in the `e2e_coverage_results` directory within your assistant's folder. The report calculates coverage separately for both passing and failing tests.

```
rasa test e2e <path-to-test-cases> --coverage-report
```

The specific output artifacts are explained below.

## Flow Coverage[​](https://rasa.com/docs/reference/testing/coverage/#flow-coverage 'Direct link to Flow Coverage')

Flow coverage report shows the percentage of flow steps that are covered by the E2E tests. Here's an example output:

```
                                           Flow Name Coverage  Num Steps  Missing Steps  Line Numbers             data/flows/order_pizza.yml::order_pizza   80.00%          5              1  [15-22]                   data/flows/add_card.yml::add_card   75.00%          4              1  [10-10]             data/flows/add_contact.yml::add_contact   25.00%          4              3  [22-35, 27-28, 31-32] data/flows/authenticate_user.yml::authenticate_user    0.00%          5              5  [11-13, 23-24, 16-24, 20-21, 14-15]         data/flows/check_balance.yml::check_balance    0.00%          2              2  [7-7, 6-6]                                               Total    40.00%        20             12
```

The `Coverage` column shows the percentage of steps covered by tests, while the `Num Steps` column indicates the total steps in each flow. The `Missing Steps` column lists steps not covered by tests, and `Line Numbers` indicates where these missing steps are located in the flow files.

important

From the above example, it is evident that majority of `data/flows/add_contact.yml::add_contact` are not well tested and none of the steps of `data/flows/authenticate_user.yml::authenticate_user` and `data/flows/check_balance.yml::check_balance` are tested. The next logical step for the user is to add E2E tests that covers the following flows.

As mentioned, the above analysis is done for both passing and failing tests and the results are also added to the `coverage_report_for_passed_tests.csv` and `coverage_report_for_failed_tests.csv` files respectively.

## Command Coverage[​](https://rasa.com/docs/reference/testing/coverage/#command-coverage 'Direct link to Command Coverage')

The tool also outputs the coverage of [commands](https://rasa.com/docs/reference/config/components/llm-command-generators/#command-reference) the dialogue understanding module triggers when running the E2E tests. The output is a histogram for passing and failing tests separately in `commands_histogram_for_passed_tests.png` and `commands_histogram_for_failed_tests.png`.

## Passing / failing tests[​](https://rasa.com/docs/reference/testing/coverage/#passing--failing-tests 'Direct link to Passing / failing tests')

Each passing and failing test is added to `passed.yml` and `failed.yml` respectively.

By using these tools, you can ensure your assistant is thoroughly tested and any gaps in coverage are identified and addressed.

- [Flow Coverage](https://rasa.com/docs/reference/testing/coverage/#flow-coverage)
- [Command Coverage](https://rasa.com/docs/reference/testing/coverage/#command-coverage)
- [Passing / failing tests](https://rasa.com/docs/reference/testing/coverage/#passing--failing-tests)