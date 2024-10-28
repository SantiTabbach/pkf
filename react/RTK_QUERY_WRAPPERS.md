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

interface BaseOptions<ResponseType, T, K> {
	onSuccess?: (response: ResponseType) => MaybePromise<T>;
	onError?: (error: unknown) => MaybePromise<K>;
	showLoader?: boolean;
}

// Mutation handler types
export type UseMutationHandlerOptions<ResponseType, S, T, K> = BaseOptions<
	ResponseType,
	S,
	T
> & {
	onFinally?: (args?: unknown) => MaybePromise<K>;
};

export type MutationHandler<
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
> = UseMutation<MutationDefinition<U, V, X, W>>;

export type QueryArg<
	U,
	V extends BaseQueryFn,
	X extends string,
	W
> = QueryArgFrom<MutationDefinition<U, V, X, W>>;

// Query handler types
export type UseQueryHandlerOptions<ResponseType, T, K> = BaseOptions<
	ResponseType,
	T,
	K
>;

export type QueryArgs<
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
> = QueryArgFrom<QueryDefinition<U, V, X, W>>;
export interface UseQueryHandlerParams<
	T,
	K,
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
> {
	query: UseQuery<QueryDefinition<U, V, X, W>>;
	options: UseQueryHandlerOptions<W, T, K>;
	queryArgs?: QueryArgs<U, V, W, X>;
	extraOptions?: QueryDefinition<U, V, X, W>['extraOptions'];
}
```

## Mutation wrapper

```typescript
import { useCallback } from 'react';
import { BaseQueryFn } from '@reduxjs/toolkit/query';
import { useDispatch } from 'react-redux';
import { setLoading } from 'src/redux/features/application/loading';
import { MutationHandler, QueryArg, UseMutationHandlerOptions } from './types';

const useMutationHandler = <
	S,
	T,
	K,
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
>(
	mutation: MutationHandler<U, V, W, X>,
	options: UseMutationHandlerOptions<W, S, T, K>
) => {
	const { onSuccess, onError, onFinally, showLoader = true } = options;

	const [mutate, status] = mutation();
	const dispatch = useDispatch();

	const wrappedMutation = useCallback(
		async (requestData: QueryArg<U, V, X, W>) => {
			showLoader && dispatch(setLoading({ isLoading: true })); // Spinner component handled by Redux, check -> https://medium.com/@santitabbach/streamlining-loading-state-management-with-redux-in-react-applications-d75765f9b224
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
				showLoader && dispatch(setLoading({ isLoading: false }));
			}
		},
		[showLoader, dispatch, mutate, onSuccess, onError, onFinally]
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
	const { wrappedMutation: dismissAlarm } = useMutationHandler(
		useDismissAlarmMutation,
		{
			onSuccess: () => {
				Toast.show({
					description: TOAST_MESSAGES.CLOSED_ALARM,
				});
			},
		}
	);

	return { dismissAlarmFunction: dismissAlarm };
};

export default useDismissAlarm;
```

## Query wrapper

```typescript
import { useEffect } from 'react';
import { BaseQueryFn } from '@reduxjs/toolkit/query';
import { useSetLoading } from 'src/redux/features/application/loading';
import { QueryArgs, UseQueryHandlerParams } from '../types';

const useQueryHandler = <
	T,
	K,
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
>({
	query,
	options,
	queryArgs = {} as QueryArgs<U, V, W, X>,
	extraOptions,
}: UseQueryHandlerParams<T, K, U, V, W, X>) => {
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
```
