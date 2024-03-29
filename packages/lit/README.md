# Lit implementation

Implementation notes:

- does not use Lit's native support for reflection (`reflect: true`) as it is asynchronous (attributes are only updated in the `update` lifecycle stage).
  Custom property accessors are used instead to explicitly (and synchronously) set the attribute.

- custom property setters also coerce the values (synchronously rather than the asynchronous nature of `willUpdate`)

- uses an internal property (here using a symbol; this could also be a private auto-accessors property when using decorators)
  rather than directly declaring the `test` property as reactive (except for the `boolean` case) to avoid triggering the specific `test` property setter when the attribute changes (which is triggered by the `test` setter), that could require a different behavior:
  the `test` setter coerces the value, validates it, then set the attribute, which triggers `attributeChangedCallback` that will set the internal property, rather than reentering the `test` setter.

- parsing of attribute values is done through a converter in the property definition,
  except for the `url` case where the document's `baseURI` could change between the attribute changed and the property is read,
  so the value is always _parsed_ in the property getter.

- default values are handled in the getter only because we already have one.
  Elements would behave the same initializing the internal property in the constructor so the property can be returned as-is from the getter,
  the internal property should only be _nullish_ on construction, and never be set back to a _nullish_ value later on
  (except for the nullable enum of course: that one starts `undefined` and could later be set to `null`, though never `undefined`).

Overall, the behavior is similar to the _cached_ variant of the [vanilla implementation](../vanilla/README.md), except using Lit's specific lifecycle hooks.
