```typescript
/**
 * Check if provided parameter is plain object
 * @param item
 * @returns boolean
 *
 * Credits: https://flowbite.com/
 */
export function isObject(item: unknown): item is Record<string, unknown> {
	return (
		item !== null && typeof item === 'object' && item.constructor === Object
	);
}
```
