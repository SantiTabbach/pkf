## Mutation wrapper

### Hook types

```typescript
import {
	BaseQueryFn,
	MutationDefinition,
	QueryArgFrom,
} from '@reduxjs/toolkit/query';
import { UseMutation } from '@reduxjs/toolkit/dist/query/react/buildHooks';
import { RequestError } from 'src/services/types';

export interface UseMutationHandlerOptions<ResponseType> {
	onSuccess?: (response: ResponseType) => Promise<void> | void;
	onError?: (error: RequestError) => Promise<void> | void;
	onFinally?: (args?: unknown) => Promise<void> | void;
	showLoader?: boolean;
}

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
```

### Hook

```typescript
import { useCallback } from 'react';
import { useDispatch } from 'react-redux';
import { BaseQueryFn } from '@reduxjs/toolkit/query';
import { setLoading } from 'src/redux/features/application/loading';
import {
	MutationHandler,
	QueryArg,
	UseMutationHandlerOptions,
} from './useMutationHandler.d.ts';

const useMutationHandler = <
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
>(
	mutation: MutationHandler<U, V, W, X>,
	options: UseMutationHandlerOptions<W>
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

### Hook types

```typescript
import {
	BaseQueryFn,
	QueryDefinition,
	QueryArgFrom,
} from '@reduxjs/toolkit/query';
import { UseQuery } from '@reduxjs/toolkit/dist/query/react/buildHooks';
import { RequestError } from '../types';

export interface UseQueryHandlerOptions<ResponseType> {
	onSuccess?: (response: ResponseType) => Promise<void> | void;
	onError?: (error: RequestError) => Promise<void> | void;
	showLoader?: boolean;
}

export interface UseQueryHandlerParams<
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
> {
	query: UseQuery<QueryDefinition<U, V, X, W>>;
	options: UseQueryHandlerOptions<W>;
	queryArgs: QueryArgFrom<QueryDefinition<U, V, X, W>>;
	extraOptions?: QueryDefinition<U, V, X, W>['extraOptions'];
}
```

### Hook

```typescript
import { useEffect } from 'react';
import { BaseQueryFn } from '@reduxjs/toolkit/query';
import { useSetLoading } from 'src/redux/features/application/loading';
import { UseQueryHandlerParams } from './useQueryHandler.d.ts';

const useQueryHandler = <
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
>({
	query,
	options,
	queryArgs = {} as UseQueryHandlerParams<U, V, W, X>['queryArgs'],
	extraOptions,
}: UseQueryHandlerParams<U, V, W, X>) => {
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
			if (isError) {
				// Perform your error enchantments here 🪄
				if (onError) {
					await onError(error);
				}
			}
		};

		void handleRequest();
	}, [data, error, isError, isLoading, isSuccess, onError, onSuccess]);

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
