# Using React Query

- Watch this video on how to use React Query: <https://youtu.be/8K1N3fE-cDs?si=M4fAV6XB2PisLPtP>

## How I used React Query

### Prerequisite - Setup Toasts

- Install [react-hot-toast](https://react-hot-toast.com/)

```bash
pnpm add react-hot-toast
```

- Setup where toasts will be shown

```tsx
// layout.tsx in the root of the app/ folder

import { Toaster } from "react-hot-toast";

export default async function RootLayout({ children }) {
  // ...

  return (
    // ...
    <Toaster />
  )
}

```

- Make sure everything works by creating a toast

```tsx
// app/login/LoginForm.tsx
"use client";

import toast from "react-hot-toast";

export default function LoginForm() {

  const showToast = () => toast.success("Toasty!")

  useEffect(() => {
    toast.success("Welcome to Timeey");
  }, []);

  return (
    <>
      <button onClick={showToast}> Click Me! </button>
    </>
  )

}
```

- Style the toast

```tsx
// layout.tsx in the root of the app/ folder

import { Toaster } from "react-hot-toast";

export default async function RootLayout({ children }) {
  // …

  return (
    // …
    <Toaster
      position="bottom-right"
      toastOptions={{
        success: {
          className: "!text-primary border-2 border-primary",
          iconTheme: {
            primary: "#635DFF",
            secondary: "#fff",
          },
        },
        error: {
          className: "!text-secondary border-2 border-secondary",
          iconTheme: {
            primary: "#FFA000",
            secondary: "#fff",
          },
        },
        loading: {
          className: "!text-black border-2 border-black",
        },
      }}
    />
  )
}
```

## Create a useTimeeyQuery hook

```ts
"use client";
import Api from "@/api/Api";
import UserCredentials from "@/models/UserCredentials";
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { AxiosError } from "axios";
import { getCookie } from "cookies-next";
import toast from "react-hot-toast";
import TimeeyError from "@/models/TimeeyError";
import ClockInRequest from "@/models/ClockInRequest";
import ClockOutRequest from "@/models/ClockOutRequest";

const parseUserCredentialsCookie = (
  userCredentialsCookie: string | undefined
) => {
  if (userCredentialsCookie === undefined) {
    return null;
  }

  return JSON.parse(userCredentialsCookie) as UserCredentials;
};

const USER_HISTORY_QUERY_KEY = "userHistory";

export default function useTimeeyQuery() {
  // get user credentials from cookie
  const userCredentials = parseUserCredentialsCookie(
    getCookie("user-credentials")
  );

  const queryClient = useQueryClient();

  // show all the errors returned by the API as toasts
  const toastApiErrors = (error: unknown) => {
    // if the error is an AxiosError, it is most likely an error from the server
    if (error instanceof AxiosError) {
      // console.log errors for debugging
      console.log(error.response?.data?.errors);
      if (error.response?.data?.errors) {
        // for each error, show a toast
        error.response?.data?.errors.forEach((error: TimeeyError) => {
          // if the error is 401, this is a special case and we want to show a custom error message
          if (error.code === 401) {
            toast.error("Unauthorized. Please login again.");
            return;
          }
          toast.error(error.message);
        });
      }
    }
  };

  // this query will run as soon as the hook is called
  const userHistoryQuery = useQuery({
    queryKey: [USER_HISTORY_QUERY_KEY],
    queryFn: () => Api.getUserHistory({ userCredentials }),
    onSuccess: () => {
      toast.success("Fetched user history");
    },
    onError: (error) => {
      console.log(error);
      toast.error("Failed to fetch user history");
      toastApiErrors(error);
    },
  });

  // these mutations do not run automatically, they have to be triggered
  const mutations = {
    createClockInRecord: useMutation({
      mutationFn: (clockInRequest: ClockInRequest) =>
        Api.createClockInRecord({ userCredentials, clockInRequest }),
      onMutate: async () => {
        // save the toast id so that we can update that specific toast
        const toastId = toast.loading("Clocking in…");
        return { toastId };
      },
      onSuccess: async (data, variables, context) => {
        await queryClient.invalidateQueries({
          queryKey: [USER_HISTORY_QUERY_KEY],
        });
        // update the loading toast
        toast.success("Clocked in successfully", { id: context?.toastId });
      },
      onError: async (error, variables, context) => {
        // update the loading toast
        toast.error("Failed to clock in", { id: context?.toastId });
        // toast api error messages
        toastApiErrors(error);
      },
    }),
    
    createClockOutRecord: useMutation({
      mutationFn: (clockOutRequest: ClockOutRequest) =>
        Api.createClockOutRecord({ userCredentials, clockOutRequest }),
      onMutate: async () => {
        // save the toast id so that we can update that specific toast
        const toastId = toast.loading("Clocking out…");
        return { toastId };
      },
      onSuccess: async (data, variables, context) => {
        await queryClient.invalidateQueries({
          queryKey: [USER_HISTORY_QUERY_KEY],
        });
        // update the loading toast
        toast.success("Clocked out successfully", { id: context?.toastId });
      },
      onError: async (error, variables, context) => {
        // update the loading toast
        toast.error("Failed to clock out", { id: context?.toastId });
        // toast api error messages
        toastApiErrors(error);
      },
    }),
  };

  return { userHistoryQuery, mutations };
}

```

- Here is how you can use this:

```tsx

"use client";
import LoadingBanana from "@/components/LoadingBanana";
import useTimeeyQuery from "@/hooks/useTimeeyQuery";
import HistoryRecord from "@/models/HistoryRecord";
import findLatestClockInRecord from "@/utils/findLatestClockInRecord";
import { redirect } from "next/navigation";
import React, { useEffect } from "react";

const TimeEntryRouter: React.FC = () => {
  const { userHistoryQuery } = useTimeeyQuery();

  useEffect(() => {
    if (userHistoryQuery.isSuccess) {
      const historyRecords = userHistoryQuery.data?.data;

      const latestClockInRecord = findLatestClockInRecord(historyRecords);

      if (latestClockInRecord) {
        redirect("/time-entry/clock-out");
      } else {
        redirect("/time-entry/clock-in");
      }
    }
  }, [userHistoryQuery]);

  if (userHistoryQuery.isError) {
    return (
      <>
        <div>Error</div>
      </>
    );
  }

  return (
    <>
      <div className="flex items-center justify-center h-full">
        <div className="w-24">
          <LoadingBanana />
        </div>
      </div>
    </>
  );
};

export default TimeEntryRouter;

```

```tsx
"use client";
import Button from "@/components/Button";
import useTimeeyQuery from "@/hooks/useTimeeyQuery";
import dayjs from "dayjs";
import { useRouter } from "next/navigation";
import React from "react";

const ClockInButton: React.FC = () => {
  // import the mutation here
  const { mutations } = useTimeeyQuery();
  const router = useRouter();

  const onClockInButtonClock = async () => {
    try {
      // that's it!
      // the toasts are already handled by the hook, so you just have to send the query!
      await mutations.createClockInRecord.mutateAsync({
        startDatetime: dayjs().toISOString(),
      });
    } finally {
      router.push("/time-entry");
    }
  };

  return (
    <>
      <Button onClick={onClockInButtonClock}>Clock In</Button>
    </>
  );
};

export default ClockInButton;

```