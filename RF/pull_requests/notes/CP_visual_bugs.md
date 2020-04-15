# RFD-3810 Fix Visual Styling Bugs

___

## Table of Contents

| Section | Name | Section | Name |
|---|---|---|---|
|I. | [Description](#description) | IV. | [Test Breaks and Fixes](#test-breaks-and-fixes) |
|II. | [Steps to Solution](#steps-to-solution) | V. | |
|III. | [Components and Their Changes](#components-and-their-changes ) | | |

___

## Description

### Problems

There are several bugs that this pull request will address. They are listed below:

- [Adjusting Margins](#adjust-margins)
- [Remove `border-left` Property](#removing-border-left)
- [Capitalize Acronyms](#capitalizing-acronyms)
- [Missing Line Break](#missing-line-break)
- [Incorrect Image Path](#incorrect-image-path)

___

#### Adjust Margins

- This bug involves the amount of non-raised space between cards in the project documentation. Prior to this PR there was a `margin-bottom: 4px` property on cards with the `rf-alert-message` class. This did not create a large enough margin. See [Solving the Margins](#solving-adjusted-margins) section for information on the fix.

___

## Steps to Solution

- [Solving the Adjusted Margins](#solving-adjused-margins)

___

### Solving Adjusted Margins

The class `rf-alert-message` has the following property:

```css
.rf-alert-message {
    margin: 0 0 4px;
}
```

#### Solution

As per the suggestion of HM, I will add the class `u-mb4` to the element with this class.`u-mbn4` provides the following property:

```css
.u-mb4 {
    margin-bottom: 16px ~important;
}
```

Thus I wil add this class to elements that are displayed in the **Provide Documentation** section.

___

## Components and Their Changes

___

## Test Breaks and Fixes

### Before Changes

Before I made any code changes the following tests failed:

#### Functional Tests : 1

- `PhantomJS 2.1.1 (Mac OS X 0.0.0) ERROR: 'Warning: Failed prop type: Object is not a constructor (evaluating 'checker(propValue, key, componentName, location, propFullName + '.' + key, ReactPropTypesSecret)')
    in Login'`


___