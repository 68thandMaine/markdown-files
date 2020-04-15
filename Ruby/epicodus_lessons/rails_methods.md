
# Rails Methods

Some of the RF applications use HTML Embeded Ruby (ERB) to display a UI. By using special tags, you can embed logic or expressions.

```ruby
<% @some_var %>
<% end %>
```

That is an execution tag.

```ruby
<%= @some_var %>
```

That is an expression tag.

___

## Methods

| Method Name | Purpose |
|---|---|
| `submit_tag()` | Creates an HTML element with the `:name` value as the displayed text in the element |
