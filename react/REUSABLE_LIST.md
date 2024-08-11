# Reusable list with custom render item component

```typescript
import { Key, ComponentType } from 'react';

/**
 * IBaseList Interface
 * @template T - Type of the individual item in the list. Can be any type, including primitives and objects.
 * @template P - Props expected by the item component. Defaults to an empty object.
 *
 * @property {T[]} items - The array of items to be rendered in the list.
 * @property {string} resourceName - The name of the prop passed to the item component, used to inject the item.
 * @property {ComponentType<P>} itemComponent - The React component used to render each item in the list.
 * @property {(item: T) => Key} [keyExtractor] - Optional function to extract the key from an item.
 */
interface IBaseList<T, P = object> {
	items: T[];
	resourceName: string;
	itemComponent: ComponentType<P>;
	keyExtractor?: (item: T) => Key;
}

/**
 * Extracts the key for an item.
 * @template T - Type of the item.
 * @param {T} item - The item to extract the key from.
 * @param {number} index - The index of the item in the list.
 * @param {(item: T) => Key} [keyExtractor] - Optional function to extract the key from the item.
 * @returns {Key} - The key to be used for the item.
 */
function getItemKey<T>(
	item: T,
	index: number,
	keyExtractor?: (item: T) => Key
): Key {
	if (keyExtractor) {
		return keyExtractor(item);
	}
	return index;
}

/**
 * BaseList Component
 * @template T - Type of the items in the list. Can be a primitive type (e.g., string) or an object.
 * @template P - Props expected by the item component.
 *
 * A generic list component that renders a list of items using the specified item component. The component is flexible
 * enough to handle items that are either objects (which might have a `key` property) or primitive types like strings
 * or numbers.
 *
 * @prop {T[]} items - The array of items to be rendered in the list.
 * @prop {string} resourceName - The name of the prop used to pass the item to the item component.
 * @prop {ComponentType<P>} itemComponent - The React component used to render each item.
 * @prop {(item: T) => Key} [keyExtractor] - Optional function to extract the key from an item.
 *
 * @returns A React element rendering the list of items.
 */
const BaseList = <T, P extends object>({
	items,
	resourceName,
	itemComponent: ItemComponent,
	keyExtractor,
}: IBaseList<T, P>) => {
	return items.map((item, index) => {
		const key = getItemKey(item, index, keyExtractor);

		return <ItemComponent key={key} {...({ [resourceName]: item } as P)} />;
	});
};

export default BaseList;
```

## Example of use

```typescript
// MediaListItem.tsx
import { sendSocialMediaEvent } from '@/utils/analytics';
import { ISocialMedia } from '@/models';
import { MediaItem } from './StyledMediaListItem';

export interface IMediaListItem {
	media: ISocialMedia;
}

const MediaListItem: React.FC<IMediaListItem> = ({ media }) => {
	const { url, title, icon: Icon } = media;

	return (
		<MediaItem
			target="_blank"
			rel="noopener noreferrer"
			key={url}
			href={url}
			title={title}
			onClick={() => sendSocialMediaEvent({ label: title })}
		>
			<Icon />
		</MediaItem>
	);
};

export default MediaListItem;

// App.tsx
import { ElementType } from 'react';
import {
	MediumIcon,
	GithubIcon,
	LinkedinIcon,
	MailIcon,
} from '@/assets/icons/socialMedia';

export interface ISocialMedia {
	icon: ElementType;
	url: string;
	title: string;
}

export const SOCIAL_MEDIA: ISocialMedia[] = [
	{
		icon: LinkedinIcon,
		title: 'LinkedIn',
		url: 'https://www.linkedin.com/in/santiago-tabbach/',
	},
	{
		icon: GithubIcon,
		title: 'GitHub',
		url: 'https://github.com/SantiTabbach',
	},
	{
		icon: MediumIcon,
		title: 'Medium',
		url: 'https://medium.com/@ymmhydfct',
	},
	{
		icon: MailIcon,
		title: 'Mail',
		url: 'mailto:tabbach97@gmail.com',
	},
];

const App = () => {
	return (
		<BaseList<ISocialMedia, IMediaListItem>
			items={SOCIAL_MEDIA}
			resourceName="media"
			itemComponent={MediaListItem}
			keyExtractor={({ title }) => title}
		/>
	);
};

export default App;
```
