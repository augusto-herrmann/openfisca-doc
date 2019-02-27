# Writing YAML tests

The recommended way to write tests is to use YAML tests.

Each formula should be tested at least with one test, and better with specific boundary values (thresholds for example).

> Terminology: Python dictionnary are called associative arrays in YAML.

## Example

In [`irpp.yaml`](https://github.com/openfisca/openfisca-france/blob/29.3.7/tests/formulas/irpp.yaml) we see:

```yaml
- name: "IRPP - Single person with salary income (1AJ) of 20 000 €"
  period: 2012
  absolute_error_margin: 0.5
  input:
    disposable_income: 20000
  output:
    irpp: -1181
```

## Common keys

- `name` (string)
- `period` (string with period syntax)
- `keywords`  (list of strings, optional)
- `description` (string, optional, multiline)
- `absolute_error_margin` (number, optional)
- `relative_error_margin` (number, optional)
- `input_variables` (associative array, keys are variable names, values are numbers)
- `output_variables` (associative array, keys are variable names, values are numbers)
- other any key defined in the model

## Syntax

### Testing formulas by giving input variables

This is the simplest way to test formulas when you only need to give input values for only one individual.

- First, name your test. Start a test with `- `, which is the YAML list separator, followed by a space, the field `name`, and the test name as a string.

```yaml
- name: "IRPP - Single person with salary income (1AJ) de 20 000 €"
```

- Then add the other relevant keys to your test. Usually, one defines the keys `period`, `keywords`, `description`, `absolute_error_margin` (or `relative_error_margin`) and their associated chosen values as follows:

```yaml
- name: "IRPP - Single person with salary income (1AJ) de 20 000 €"
  period: 2012
  absolute_error_margin: 0.5
```

- Create nested dictionnaries within the keys `input_variables` and `output_variables`,
which keys are variable names and values are numbers, respectively input and expected values.
For instance:

```yaml
- name: "IRPP - Single person with salary income (1AJ) de 20 000 €"
  period: 2012
  absolute_error_margin: 0.5
  input:
    disposable_income: 20000
    gross_income: 20000
  output:
    irpp: -1181
```


### Testing formulas giving a test case

This is the simplest way to test formulas when you need to give input values for many individuals
which are dispatched into entities.

> See the last test of [cotisations_sociales_simulateur_IPP.yaml](https://github.com/openfisca/openfisca-france/blob/29.3.7/tests/cotisations_sociales_simulateur_IPP.yaml#L244-L303)

In this case, there is another convention:

- do not include the field `input_variables` but instead define new keys corresponding to the entities:

    ```yaml
    - name: "IRPP - Family with salary income of 20,000 €"
    period: 2012
    absolute_error_margin: 0.5
    input:
      families:
      households:
      tax_homes:
    ```

- define the individuals with their `id` and their variables:

    ```yaml
    individuals:
      parent1:
        date_of_birth: 1972-01-01
        depcom_entreprise: "69381"
        primes_fonction_publique: 500
      parent2:
        date_of_birth: 1972-01-01
        depcom_entreprise: "69381"
        primes_fonction_publique: 500
        traitement_indiciaire_brut: 2000
      child1:
        date_of_birth: 2000-01-01
      child2:
        date_of_birth: 2009-01-01
    ```

- specify the relations between individuals and their entity:

    ```yaml
    families:
        parents: ["parent1", "parent2"]
        children: ["child1", "child2"]
    households:
        reference_person: "parent1"
        conjoint: "parent2"
        children: ["child1", "child2"]
    tax_homes:
        registrants: ["parent1", "parent2"]
        dependents: ["child1", "child2"]
    ```

- finally, define a dictionnary of the expected values of the output variables. Each output variable takes a list of length equal to the number of individuals defined in the test. E.g, for a family of four individuals with two working parents and two unemployed children, the output variable superannuation_contribution is defined as follows:

    ```yaml
    output:
        superannuation_contribution: [3500, 2500, 0, 0]
    ```

### Testing formulas using variables defined for multiple periods

Input or output variables can be defined for multiple periods by giving an associated array
which keys are a period expression and values are the value for that period.

Values can be arithmetic expressions too.

```yaml
  individuals:
    base_salary:
      2013-01: 35 * 52 / 12 * 9
      2013-02: 35 * 52 / 12 * 9
      2013-03: 35 * 52 / 12 * 9
```

## Running a test

To run YAML tests, use the command line tool `openfisca test`, documented [here](../../openfisca-python-api/openfisca-run-test.html):

```sh
openfisca test path/to/file.yaml
```

## Next steps

Other kinds of tests exist, see [contribute/tests](../contribute/tests.md).