## Wrapper types

```typescript
import {
	BaseQueryFn,
	MutationDefinition,
	QueryArgFrom,
	QueryDefinition,
} from '@reduxjs/toolkit/query';
import {
	UseMutation,
	UseQuery,
} from '@reduxjs/toolkit/dist/query/react/buildHooks';
import { MaybePromise } from '@reduxjs/toolkit/dist/query/tsHelpers';

interface BaseOptions<ResponseType, S, T> {
	onSuccess?: (response: ResponseType) => MaybePromise<T>;
	onError?: (error: unknown) => MaybePromise<S>;
	showLoader?: boolean;
}

// Mutation handler types
export type MutationArgs<
	U,
	V extends BaseQueryFn,
	X extends string,
	W
> = QueryArgFrom<MutationDefinition<U, V, X, W>>;

export interface UseMutationHandlerParams<
	R,
	S,
	T,
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
> {
	mutation: UseMutation<MutationDefinition<U, V, X, W>>;
	options?: BaseOptions<W, S, T> & {
		onFinally?: (args?: unknown) => MaybePromise<R>;
	};
}

// Query handler types
export type QueryArgs<
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
> = QueryArgFrom<QueryDefinition<U, V, X, W>>;

export interface UseQueryHandlerParams<
	S,
	T,
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
> {
	query: UseQuery<QueryDefinition<U, V, X, W>>;
	options?: BaseOptions<W, S, T>;
	queryArgs?: QueryArgs<U, V, W, X>;
	extraOptions?: QueryDefinition<U, V, X, W>['extraOptions'];
}
```

## Mutation wrapper

```typescript
import { useCallback } from 'react';
import { BaseQueryFn } from '@reduxjs/toolkit/query';
import { useSetLoading } from 'src/redux/features/application/loading';
import { MutationArgs, UseMutationHandlerParams } from './types';

const useMutationHandler = <
	R,
	S,
	T,
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
>({
	mutation,
	options = {},
}: UseMutationHandlerParams<R, S, T, U, V, W, X>) => {
	const { onSuccess, onError, onFinally, showLoader = true } = options;

	const [mutate, status] = mutation();

	useSetLoading(showLoader && status.isLoading); // Spinner component handled by Redux, check -> https://medium.com/@santitabbach/streamlining-loading-state-management-with-redux-in-react-applications-d75765f9b224

	const wrappedMutation = useCallback(
		async (requestData: MutationArgs<U, V, X, W>) => {
			try {
				const response = await mutate(requestData).unwrap();

				// Perform your success enchantments here 🪄
				if (onSuccess) {
					return await onSuccess(response);
				}

				return response;
			} catch (error) {
				// Perform your error enchantments here 🪄
				if (onError) {
					return await onError(error);
				}
			} finally {
				// Perform your final enchantments here 🪄
				if (onFinally) {
					await onFinally();
				}
			}
		},
		[mutate, onSuccess, onError, onFinally]
	);

	return { wrappedMutation, ...status };
};

export default useMutationHandler;
```

### Example of use

```typescript
import { useMutationHandler } from 'src/services';
import { useDismissAlarmMutation } from 'src/redux/features/alertifyCoreApi/alarms';
import { TOAST_MESSAGES, Toast } from 'src/application/components';

const useDismissAlarm = () => {
	const { wrappedMutation: dismissAlarm } = useMutationHandler({
		mutation: useDismissAlarmMutation,
		options: {
			showLoader: false,
			onSuccess: () => {
				Toast.show({
					description: TOAST_MESSAGES.CLOSED_ALARM,
				});
			},
		},
	});

	return dismissAlarm;
};

export default useDismissAlarm;

const CloseAlarmModal = ({ isVisible, setIsVisible, alarmId }: Props) => {
    const [description, setDescription] = useState(EMPTY_STRING);

    const dismissAlarm = useDismissAlarm();

    const handleSubmit = async () => {
      setIsVisible(false);

	  await dismissAlarm({ id: alarmId, description });
    };

    return (...)
  };

  export default CloseAlarmModal;
```

## Query wrapper

```typescript
import { useEffect } from 'react';
import { BaseQueryFn } from '@reduxjs/toolkit/query';
import { useSetLoading } from 'src/redux/features/application/loading';
import { QueryArgs, UseQueryHandlerParams } from './types';

const useQueryHandler = <
	S,
	T,
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
>({
	query,
	options = {},
	queryArgs = {} as QueryArgs<U, V, W, X>,
	extraOptions,
}: UseQueryHandlerParams<S, T, U, V, W, X>) => {
	const { onSuccess, onError, showLoader = true } = options;

	const {
		isSuccess,
		isError,
		data,
		error,
		isLoading,
		isFetching,
		...queryResult
	} = query(queryArgs, extraOptions);

	useSetLoading(showLoader && (isFetching || isLoading));

	useEffect(() => {
		const handleRequest = async () => {
			if (isSuccess) {
				// Perform your success enchantments here 🪄
				if (onSuccess) {
					await onSuccess(data);
				}
			}
			if (isError && onError) {
				if (onError) {
					// Perform your error enchantments here 🪄
					if (onError) {
						await onError(error);
					}
				}
			}
		};

		void handleRequest();
	}, [data, error, isError, isSuccess, onError, onSuccess]);

	return {
		isSuccess,
		isError,
		data,
		error,
		isLoading,
		isFetching,
		...queryResult,
	};
};

export default useQueryHandler;
```

### Example of use

```typescript
import { useGetActiveNeighborhoodsQuery } from 'src/redux/features/alertifyCoreApi/map/apiMapSlice';
import { useQueryHandler } from 'src/services';

const useGetActiveNeighborhoods = () => {
	const { data: activeNeighborhoods = [] } = useQueryHandler({
		query: useGetActiveNeighborhoodsQuery,
		options: {
			showLoader: false,
		},
		extraOptions: {
			refetchOnMountOrArgChange: true,
		},
	});

	return { activeNeighborhoods };
};

export default useGetActiveNeighborhoods;

const MapComponent = () => {
  const { colorMode } = useColorMode();

  const { activeNeighborhoods } = useGetActiveNeighborhoods();

  return (...);
};

export default MapComponent;

```
