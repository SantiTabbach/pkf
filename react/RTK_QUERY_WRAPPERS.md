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

/**
 * BaseOptions: Define optional callbacks for handling
 * success, error, and loading states for both mutations and queries.
 *
 * @template ResponseType - The expected type of the API response.
 * @template S - The type of the error state.
 * @template T - The type of the success state.
 *
 * @property {function} [onSuccess] - A callback executed on a successful response.
 * @property {function} [onError] - A callback executed on an error.
 * @property {boolean} [showLoader] - Whether to display a loading spinner (default: true).
 */
interface BaseOptions<ResponseType, S, T> {
	onSuccess?: (response: ResponseType) => MaybePromise<T>;
	onError?: (error: unknown) => MaybePromise<S>;
	showLoader?: boolean;
}

// Mutation handler types

/**
 * MutationArgs: Defines the arguments for mutation functions.
 *
 * @template U - The mutation argument type.
 * @template V - BaseQueryFn from Redux Toolkit for managing base queries.
 * @template X - The cache key type for the mutation.
 * @template W - The response type for the mutation.
 */
export type MutationArgs<
	U,
	V extends BaseQueryFn,
	X extends string,
	W
> = QueryArgFrom<MutationDefinition<U, V, X, W>>;

/**
 * UseMutationHandlerParams: Defines the parameters for the mutation handler hook.
 *
 * @template R - Type of the 'finally' callback return.
 * @template S - Type of the 'onError' callback return.
 * @template T - Type of the 'onSuccess' callback return.
 * @template U - The mutation argument type.
 * @template V - BaseQueryFn from Redux Toolkit for managing base queries.
 * @template W - The response type for the mutation.
 * @template X - The cache key type for the mutation.
 *
 * @property {UseMutation} mutation - The mutation hook from Redux Toolkit.
 * @property {object} [options] - Optional callbacks for success, error, and finally.
 */
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

/**
 * QueryArgs: Defines the arguments for query functions.
 *
 * @template U - The query argument type.
 * @template V - BaseQueryFn from Redux Toolkit for managing base queries.
 * @template W - The response type for the query.
 * @template X - The cache key type for the query.
 */
export type QueryArgs<
	U,
	V extends BaseQueryFn,
	W,
	X extends string = string
> = QueryArgFrom<QueryDefinition<U, V, X, W>>;

/**
 * UseQueryHandlerParams: Defines the parameters for the query handler hook.
 *
 * @template S - Type of the 'onError' callback return.
 * @template T - Type of the 'onSuccess' callback return.
 * @template U - The query argument type.
 * @template V - BaseQueryFn from Redux Toolkit for managing base queries.
 * @template W - The response type for the query.
 * @template X - The cache key type for the query.
 *
 * @property {UseQuery} query - The query hook from Redux Toolkit.
 * @property {object} [options] - Optional callbacks for success, error.
 * @property {object} [extraOptions] - Extra options for query.
 */
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

/**
 * useMutationHandler: A hook that wraps the RTK Query mutation hook to handle loading,
 * success, error, and finally callbacks in a unified and extensible way.
 *
 * @template R - The return type for the 'finally' callback.
 * @template S - The return type for the 'onError' callback.
 * @template T - The return type for the 'onSuccess' callback.
 * @template U - The mutation argument type.
 * @template V - BaseQueryFn from Redux Toolkit for managing base queries.
 * @template W - The response type for the mutation.
 * @template X - The cache key type for the mutation.
 *
 * @param {UseMutationHandlerParams<R, S, T, U, V, W, X>} params - Contains the mutation function and optional callbacks.
 *
 */
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

// useCreateUser.ts
import { useMutationHandler } from 'src/services';
import { useCreateUserMutation } from 'src/redux/features/users/apiUsers';
import { TOAST_MESSAGES, Toast } from 'src/application/components';

const useCreateUser = () => {
	const { wrappedMutation: createUser } = useMutationHandler({
		mutation: useCreateUserMutation,
		options: {
			showLoader: false,
			onSuccess: () => {
				Toast.show({
					description: TOAST_MESSAGES.CREATE_USER,
				});
			},
		},
	});

	return createUser;
};

export default useCreateUser;

// UserForm.tsx
const UserForm = ({ userPayload }: { userPayload: UserPayload }) => {
    const createUser = useCreateUser();

    const handleSubmit = async () => {
	  await createUser(userPayload);
    };

    return <Form onSubmit={handleSubmit} ... />
  };

  export default UserForm;
```

## Query wrapper

```typescript
import { useEffect } from 'react';
import { BaseQueryFn } from '@reduxjs/toolkit/query';
import { useSetLoading } from 'src/redux/features/application/loading';
import { QueryArgs, UseQueryHandlerParams } from './types';

/**
 * useQueryHandler: A hook that wraps the RTK Query query hook to handle loading,
 * success, and error callbacks in a unified and extensible way.
 *
 * @template S - Type for the 'onError' callback.
 * @template T - Type for the 'onSuccess' callback.
 * @template U - The query argument type.
 * @template V - BaseQueryFn from Redux Toolkit for managing base queries.
 * @template W - The response type for the query.
 * @template X - The cache key type for the query.
 *
 * @param {UseQueryHandlerParams<S, T, U, V, W, X>} params - Contains the query function and optional callbacks.
 *
 */
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

// useGetUsers.ts
import { useGetUsersQuery } from 'src/redux/features/users/apiUsers';
import { useQueryHandler } from 'src/services';

const useGetUsers = () => {
	const { data: users = [], ...rest } = useQueryHandler({
		query: useGetUsersQuery,
		options: {
			showLoader: false,
		},
		extraOptions: {
			refetchOnMountOrArgChange: true,
		},
	});

	return { users, ...rest };
};

export default useGetUsers;

// UsersList.tsx
const UsersList = () => {
  const { users, error } = useGetUsers();

  if(error) {
	return <Error />
  }

  return users.map(...);
};

export default UsersList;

```
