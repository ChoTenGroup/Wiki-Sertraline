---
icon: desktop-arrow-down
---

# Section: Custom Data

## Custom Data Structure Usage

{% hint style="info" %}
Before and including 1.20.4, Custom Data is used for custom NBT data. In and after 1.20.5, Custom Data is used for the custom\_data data component.
{% endhint %}

```plaintext
sample_item:
  custom_data:
    custom_value: hello
    some_number: 42
```

Before and including 1.20.4, this will create two NBT entries: custom\_value: hello, some\_number: 42. In and after 1.20.5, this will place custom\_value: hello, some\_number: 42 inside the custom\_data data component.

Deep nesting, maps, and lists are supported.

```plaintext
sample_item:
  custom_data:
    deep:
      aaa: 1
    sample_list:
      - aaa
      - bbb
    sample_map_list:
       - aa: aa
       - bb: bb
```
