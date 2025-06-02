```typescript
import { useState } from 'react';
import {
	createPersistData,
	createRetrieveData,
	LS_KEYS,
} from './utils/local-storage';

interface Args<T> {
	key: LS_KEYS;
	initialValue?: T;
}

const usePersistedState = <T>({ key, initialValue }: Args<T>) => {
	const persistOnStorage = createPersistData(key, false);
	const retrieveFromStorage = createRetrieveData(key);

	const [value, setValue] = useState(() => {
		return retrieveFromStorage() ?? initialValue;
	});

	const setPersistedValue = (value: T) => {
		setValue(value);

		persistOnStorage(JSON.stringify(value));
	};

	return [value, setPersistedValue] as const;
};

export default usePersistedState;

// Example of use

const ThemeToggle = () => {
	const [value, setValue] = usePersistedState({
		key: LS_KEYS.THEME,
		initialValue: 'dark',
	});

	return (
		<div>
			<p> Current theme: {value}</p>
			<button onClick={() => setValue('light')}>Set theme to light</button>
			<button onClick={() => setValue('dark')}>Set theme to dark</button>
		</div>
	);
};
```
