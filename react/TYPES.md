```typescript
/**
 * Defines a prop that allows specifying a custom component type (`as` prop).
 *
 * @template C - The React element type to be used.
 */
export type AsProp<C extends React.ElementType> = {
	as?: C | null;
};

/**
 * Determines which props should be omitted when spreading props to a component.
 *
 * @template C - The React element type.
 * @template P - The additional props.
 */
export type PropsToOmit<C extends React.ElementType, P> = keyof (AsProp<C> & P);

/**
 * Defines the props for a polymorphic component, allowing it to accept a custom `as` prop
 * and inherit the appropriate attributes from the specified element type.
 *
 * @template C - The React element type.
 * @template Props - Additional component props.
 */
export type PolymorphicComponentProp<
	C extends React.ElementType,
	Props = object
> = React.PropsWithChildren<Props & AsProp<C>> &
	Omit<React.ComponentPropsWithoutRef<C>, PropsToOmit<C, Props>>;

/**
 * Extends `PolymorphicComponentProp` to include a `ref` for forwarding references.
 *
 * @template C - The React element type.
 * @template Props - Additional component props.
 */
export type PolymorphicComponentPropWithRef<
	C extends React.ElementType,
	Props = object
> = PolymorphicComponentProp<C, Props> & {
	ref?: PolymorphicRef<C>;
};

/**
 * Extracts the `ref` type for a given React element type.
 *
 * @template C - The React element type.
 */
export type PolymorphicRef<C extends React.ElementType> =
	React.ComponentPropsWithRef<C>['ref'];
```
