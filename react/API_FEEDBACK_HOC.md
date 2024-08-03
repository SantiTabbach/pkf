### Custom hook to handle API feedback

```typescript
/**
 * Custom hook to handle API feedback states.
 *
 * @param {FetchFunction<T>} fetchFunction - Function that fetches data from an API.
 * @param {HocConfig} [config=defaultHocConfig] - Configuration object for customizing error, empty, and loading props.
 * @returns {{ Component: Component | null, props?: any, data?: T }} - The feedback component and its props or fetched data.
 */

import { FetchError, EmptyData, Loading } from '@/your/components/path';
import { FetchFunction, HocConfig } from '@/your/models/path';

const defaultHocConfig: HocConfig = {
	errorProps: { message: 'Something went wrong.' },
	emptyProps: { message: 'Nothing to show.' },
	loadingProps: { message: 'loading...' },
};

const useApiFeedback = <T>(
	fetchFunction: FetchFunction<T>,
	config: HocConfig = defaultHocConfig
) => {
	const { data, isError, isLoading } = fetchFunction();

	if (isLoading) {
		return {
			Component: Loading,
			props: { ...defaultHocConfig.loadingProps, ...config.loadingProps },
		};
	}

	if (isError) {
		return {
			Component: FetchError,
			props: { ...defaultHocConfig.errorProps, ...config.errorProps },
		};
	}

	if (!data) {
		return {
			Component: EmptyData,
			props: { ...defaultHocConfig.emptyProps, ...config.emptyProps },
		};
	}

	return { Component: null, data };
};

export default useApiFeedback;
```

### Higher Order Component that wrap the component

```typescript
/**
 * Higher-Order Component (HOC) that provides feedback handling for API calls.
 * It wraps a component and manages loading, error, and empty state feedback.
 *
 * @param {Component<P>} WrappedComponent - Component to be wrapped.
 * @param {FetchFunction<T>} fetchFunction - Function that fetches data from an API.
 * @param {string} [resourceName='data'] - Name of the resource to pass as prop to the wrapped component.
 * @param {HocConfig} [hocConfig] - Configuration object for customizing error, empty, and loading props.
 * @returns {Component<P>} - Component with API feedback handling.
 */

import React, { ReactElement } from 'react';
import { useApiFeedback } from '@/your/hooks/path';
import { Component, FetchFunction, HocConfig } from '@/your/models/path';

const withApiFeedback = <P, T>(
	WrappedComponent: Component<P>,
	fetchFunction: FetchFunction<T>,
	resourceName = 'data',
	hocConfig?: HocConfig
): React.FC<P> => {
	return (props: P): ReactElement => {
		const {
			Component,
			props: feedbackProps,
			data,
		} = useApiFeedback(fetchFunction, hocConfig);

		if (Component) {
			return <Component {...feedbackProps} />;
		}

		const resourceProps = {
			[resourceName]: data,
		};

		return <WrappedComponent {...props} {...resourceProps} />;
	};
};

export default withApiFeedback;
```

### Example of types

```typescript
import { ReactElement } from 'react';
import { FetchError, EmptyData, Loading } from '@/your/components/path';

export type FetchFunction<T> = () => {
	data: T;
	isError: boolean;
	isLoading: boolean;
}; // Your fetch type

export interface HocConfig {
	errorProps?: React.ComponentProps<typeof FetchError>;
	emptyProps?: React.ComponentProps<typeof EmptyData>;
	loadingProps?: React.ComponentProps<typeof Loading>;
}

export type Component<P> = (props: P) => ReactElement;
```

## Example of use

```typescript
import React from 'react';
import withApiFeedback from './withApiFeedback';

// Example fetch function that simulates an API call
const fetchUserData = () => {
	const isLoading = false;
	const isError = false;
	const data = { name: 'John Doe', age: 30 }; // Example data

	return { data, isError, isLoading };
};

// Example component that will receive the data as a prop
interface UserComponentProps {
	data: { name: string; age: number };
}

const UserComponent: React.FC<UserComponentProps> = ({ data }) => {
	return (
		<div>
			<h1>User Info</h1>
			<p>Name: {data.name}</p>
			<p>Age: {data.age}</p>
		</div>
	);
};

// Wrap the UserComponent with the withApiFeedback HOC
const UserComponentWithFeedback = withApiFeedback(UserComponent, fetchUserData);

// Render the wrapped component in your application
const App: React.FC = () => {
	return (
		<div className="App">
			<UserComponentWithFeedback />
		</div>
	);
};

export default App;
```
