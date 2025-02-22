---
id: api
title: API
sidebar_label: API
---

`Angular Testing Library` re-exports everything from `DOM Testing Library` as
well as these methods:

- [`render`](<(#render)>)

## `render`

```typescript
async function render<ComponentType>(
  component: Type<ComponentType>,
  renderOptions?: RenderComponentOptions<ComponentType>
): Promise<RenderResult<ComponentType, ComponentType>>
async function render<DirectiveType, WrapperType = WrapperComponent>(
  component: Type<DirectiveType>,
  renderOptions?: RenderDirectiveOptions<DirectiveType, WrapperType>
): Promise<RenderResult<DirectiveType, WrapperType>>
```

## Component RenderOptions

### `detectChanges`

Will call `detectChanges` when the component is compiled

**default** : `true`

**example**:

```typescript
const component = await render(AppComponent, { detectChanges: false })
```

### `declarations`

A collection of providers needed to render the component via Dependency
Injection, for example, injectable services or tokens.

For more info see the
[Angular docs](https://angular.io/api/core/NgModule#providers).

**default** : `[]`

**example**:

```typescript
const component = await render(AppComponent, {
  providers: [
    CustomersService,
    {
      provide: MAX_CUSTOMERS_TOKEN,
      useValue: 10,
    },
  ],
})
```

### `imports`

A collection of imports needed to render the component, for example, shared
modules. Adds `NoopAnimationsModule` by default if `BrowserAnimationsModule`
isn't added to the collection

For more info see the
[Angular docs](https://angular.io/api/core/NgModule#imports).

**default** : `[NoopAnimationsModule]`

**example**:

```typescript
const component = await render(AppComponent, {
  imports: [AppSharedModule, MaterialModule],
})
```

### `schemas`

A collection of schemas needed to render the component. Allowed values are
`NO_ERRORS_SCHEMA` and `CUSTOM_ELEMENTS_SCHEMA`.

For more info see the
[Angular docs](https://angular.io/api/core/NgModule#schemas).

**default** : `[]`

**example**:

```typescript
const component = await render(AppComponent, {
  schemas: [NO_ERRORS_SCHEMA],
})
```

### `componentProperties`

An object to set `@Input` and `@Output` properties of the component.

**default** : `{}`

**example**:

```typescript
const component = await render(AppComponent, {
  componentProperties: {
    counterValue: 10,
    send: (value) => { ... }
  }
})
```

### `componentProviders`

A collection of providers to inject dependencies of the component.

For more info see the
[Angular docs](https://angular.io/api/core/Directive#providers).

**default** : `[]`

**example**:

```typescript
const component = await render(AppComponent, {
  componentProviders: [AppComponentService],
})
```

### `queries`

Queries to bind. Overrides the default set from DOM Testing Library unless
merged.

**default** : `undefined`

**example**:

```typescript
const component = await render(AppComponent, {
  queries: { ...queries, ...customQueries },
})
```

### `excludeComponentDeclaration`

Exclude the component to be automatically be added as a declaration. This is
needed when the component is declared in an imported module.

**default** : `false`

**example**:

```typescript
const component = await render(AppComponent, {
  imports: [AppModule], // a module that includes AppComponent
  excludeComponentDeclaration: true,
})
```

### `routes`

The route configuration to set up the router service via
`RouterTestingModule.withRoutes`. For more info see the
[Angular Routes docs](https://angular.io/api/router/Routes).

**default** : `[]`

**example**:

```typescript
const component = await render(AppComponent, {
  declarations: [ChildComponent],
  routes: [
    {
      path: '',
      children: [
        {
          path: 'child/:id',
          component: ChildComponent,
        },
      ],
    },
  ],
})
```

## Directive RenderOptions

To test a directive, the render API is a bit different. The API has the same
options as the Component RenderOptions, but has more options:

### `template`

The template to render the directive.

**example**:

```typescript
const component = await render(SpoilerDirective, {
  template: `<div spoiler message='SPOILER'></div>`,
})
```

### `wrapper`

An Angular component to wrap the directive in.

**default**: `WrapperComponent` , an empty component that strips the
`ng-version` attribute.

**example**:

```typescript
const component = await render(SpoilerDirective, {
  template: `<div spoiler message='SPOILER'></div>`
  wrapper: CustomWrapperComponent
})
```

## `RenderResult`

### `...queries`

The most important feature of `render` is that the queries from
[DOM Testing Library](https://testing-library.com/docs/dom-testing-library) are
automatically returned with their first argument bound to the component under
test.

See [Queries](https://testing-library.com/docs/dom-testing-library/api-queries)
for a complete list.

**example**:

```typescript
const component = await render(AppComponent)

component.getByText('Hello world')
component.queryByLabelText('First name:')
```

### `fireEvent.***`

The second most important feature of `render` is that the events from
[DOM Testing Library](https://testing-library.com/docs/dom-testing-library) are
automatically bound to the Angular Component. This to ensure that the Angular
change detection has been run by calling `detectChanges` after an event has been
fired.

See
[Firing Events](https://testing-library.com/docs/dom-testing-library/api-events)
for a complete list.

**example**:

```typescript
const component = await render(AppComponent)

component.change(component.getByLabelText('Username'), {
  target: {
    value: 'Tim',
  },
})
component.click(component.getByText('Login'))
```

### `type`

Types a value in an input field with the same interactions as the user would do.

```typescript
const component = await render(AppComponent)

component.type(component.getByLabelText('Username'), 'Tim')
component.type(component.getByLabelText('Username'), 'Tim', { delay: 250 })
```

### `selectOptions`

Select an option(s) from a select field with the same interactions as the user
would do.

```typescript
const component = await render(AppComponent)

component.selectOptions(component.getByLabelText('Fruit'), 'Blueberry')
component.selectOptions(component.getByLabelText('Fruit'), ['Blueberry'. 'Grape'])
```

### `navigate`

Accepts a DOM element or a path as parameter. If an element is passed,
`navigate` will navigate to the `href` value of the element. If a path is
passed, `navigate` will navigate to the path.

```typescript
const component = await render(AppComponent, {
  routes: [...]
})

await component.navigate(component.getByLabelText('To details'))
await component.navigate('details/3')
```

### `fixture`

The Angular `ComponentFixture` of the component.

For more info see the
[Angular docs](https://angular.io/api/core/testing/ComponentFixture).

```typescript
const component = await render(AppComponent)

const componentInstance = component.fixture.componentInstance as AppComponent
```

> 🚨 If you find yourself using `fixture` to access the component's internal
> values you should reconsider! This probable means, you're testing
> implementation details.

### `container`

The containing DOM node of your rendered Angular Component. This is a regular
DOM node, so you can call `container.querySelector` etc. to inspect the
children.

### `debug`

Prints out the component's DOM with syntax highlighting. Accepts an optional
parameter, to print out a specific DOM node.

```typescript
const component = await render(AppComponent)

component.debug()
component.debug(component.getByRole('alert'))
```
