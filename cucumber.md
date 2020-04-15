# Cucumber Stuff

## Features

Cucumber is a BDD testing framework that uses specific language to define what we want the software to do.

Each example that is defined is called a scenario. Scenarios are defined in a dir called `/feature` and end with `.feature`.

> `features/is_it_friday.feature`

Within this file let's imagine the following text:

```feature
Feature: Is it Friday yet?
  Everybody wants to know when it's Friday

  Scenario: Sunday isn't Friday
    Given today is Sunday
    When I ask whether it's Friday yet
    Then I should be told "Nope"
```

This block has several keywords that are picked up on by Cucumber:

- **Feature:**
  
  - The feature keyword provides a high level description of the software feature, and to group related scenarios. It is always follwed by some text that should match the file name.

- **Scenario**

  - The scenario keyword is a concrete example that illustrates a business rule and consists of a list of steps.

- **Scenario Outline**

  - Scenario Outlines are

- **Steps**

  - Steps are executed one at a time in the order in which they are defined. The following keywords are all steps:

    - **Given**
      - Describes the initial context of the system (the scene of the scenario / typically something that has happened in the past).
    - **When**
      - Describes an event or an action. (person or system interacting with the system).
    - **Then**
      - Used to describe an expected outcome.
    - **And/But**
      - Used if there are successive steps.

Cucumber knows how to use these values to print test skeletons if you run `$ cucumber`. It's a pretty neat way to avoid having to write your own test scenarios in a  file called `stepdefs.rb`.

___
___

## Stepdefs

Files are located in `./features/step_definitions` and are used to write tests using the `.feature` extension files.

It seems like varibles are created in each block and given to the seceding block until. For example in the above text the following is written in the test blocks

### Given

```ruby
Given("today is Sunday") do
  @today = 'Sunday'
end
```

### When

```ruby
When("I ask whether it's Friday yet") do
  @actual_answer = is_it_friday(@today)
end
```

- The actual answer uses a [Helper Module](#helper-module). This could be the method that is being tested.

### Then

```ruby
Then("I should be told {string}") do |expected_answer|
  expect(@actual_answer).to eq(expected_answer)
end
```

- Is this string interpolation where `{string}` == text declared in the feature?

- Is the `expected_answer` the value of `{string}`?

- In the above example it looks like "Nope" is in double quotes. Perhaps it is string interpolation in a way.

___

#### Note that the actual answer is declared in the when statement.

#### Note that the value that is is used to create the actual answer is declared in the Given block.

___
___

