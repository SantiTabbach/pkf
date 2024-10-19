## Mutation wrapper

```typescript
import { useCallback } from 'react';
import { useDispatch } from 'react-redux';
import {
	BaseQueryFn,
	MutationDefinition,
	QueryArgFrom,
} from '@reduxjs/toolkit/query';
import { UseMutation } from '@reduxjs/toolkit/dist/query/react/buildHooks';
import { setLoading } from 'src/redux/features/application/loading';

interface UseMutationHandlerOptions<ResponseType> {
	onSuccess?: (response: ResponseType) => void;
	onError?: (error: unknown) => void;
	onFinally?: (args?: unknown) => void;
	showLoader?: boolean;
}

const useMutationHandler = <
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
>(
	mutation: UseMutation<MutationDefinition<U, V, X, W>>,
	options: UseMutationHandlerOptions<W>
) => {
	const [mutate, status] = mutation();

	const { onSuccess, onError, onFinally, showLoader = true } = options;
	const dispatch = useDispatch();

	const wrappedMutation = useCallback(
		async (requestData: QueryArgFrom<MutationDefinition<U, V, X, W>>) => {
			showLoader && dispatch(setLoading({ isLoading: true })); // Spinner component handled by Redux, check -> https://medium.com/@santitabbach/streamlining-loading-state-management-with-redux-in-react-applications-d75765f9b224
			try {
				const response = await mutate(requestData).unwrap();
				// Perform your success enchantments here 🪄
				if (onSuccess) {
					return onSuccess(response);
				}

				return response;
			} catch (error) {
				// Perform your error enchantments here 🪄
				if (onError) {
					return onError(error);
				}
			} finally {
				// Perform your final enchantments here 🪄
				if (onFinally) {
					onFinally();
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
import {
	BaseQueryFn,
	QueryDefinition,
	QueryArgFrom,
} from '@reduxjs/toolkit/query';
import { UseQuery } from '@reduxjs/toolkit/dist/query/react/buildHooks';
import { useSetLoading } from 'src/redux/features/application/loading';
import { RequestError } from '../types';
import { REQUEST_ACTION } from '../enums';

interface UseQueryHandlerOptions<ResponseType> {
	onSuccess?: (response: ResponseType) => void;
	onError?: (error: RequestError) => void;
	onFinally?: () => void;
	showLoader?: boolean;
}

interface UseQueryHandlerParams<
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
> {
	query: UseQuery<QueryDefinition<U, V, X, W>>;
	options: UseQueryHandlerOptions<W>;
	queryArgs?: QueryArgFrom<QueryDefinition<U, V, X, W>>;
	extraOptions?: QueryDefinition<U, V, X, W>['extraOptions'];
}

const useQueryHandler = <
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
>({
	query,
	options,
	queryArgs,
	extraOptions,
}: UseQueryHandlerParams<U, V, W, X>) => {
	const { onSuccess, onError, onFinally, showLoader = true } = options;

	const {
		isSuccess,
		isError,
		data,
		error,
		isLoading,
		isFetching,
		...queryResult
	} = query(queryArgs!, extraOptions);

	useSetLoading(showLoader && (isFetching || isLoading)); // Spinner component handled by Redux, check -> https://medium.com/@santitabbach/streamlining-loading-state-management-with-redux-in-react-applications-d75765f9b224

	useEffect(() => {
		if (isSuccess) {
			// Perform your success enchantments here 🪄
			if (onSuccess) {
				onSuccess(data);
			}
		}
		if (isError) {
			// Perform your error enchantments here 🪄
			if (onError) {
				onError(error);
			}
		}

		if (!isLoading && onFinally) {
			// Perform your final enchantments here 🪄
			onFinally();
		}
	}, [
		data,
		error,
		isError,
		isLoading,
		isSuccess,
		onError,
		onFinally,
		onSuccess,
	]);

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
import { REQUEST_ACTION, useQueryHandler } from 'src/services';

const { FETCH_ACTIVE_NEIGHBORHOODS } = REQUEST_ACTION;

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
