# Setup API

- My API file looks like this:

```ts

import ClockInRequest from "@/models/ClockInRequest";
import ClockOutRequest from "@/models/ClockOutRequest";
import HistoryRecord from "@/models/HistoryRecord";
import UserCredentials from "@/models/UserCredentials";
import Username from "@/models/Username";
import axios, { AxiosResponse } from "axios";

const axiosInstance = axios.create({
  baseURL: "http://localhost:8000/api/v1",
});

const createAuthorizationHeader = (userCredentials: UserCredentials) => {
  return `Basic ${btoa(
    `${userCredentials.username}:${userCredentials.password}`
  )}`;
};

const setAuthorizationHeader = (userCredentials: UserCredentials | null) => {
  if (!userCredentials) {
    axiosInstance.defaults.headers.common["Authorization"] = null;
    return;
  }

  axiosInstance.defaults.headers.common["Authorization"] =
    createAuthorizationHeader(userCredentials);
};

class Api {
  static async getAllUsernames(): Promise<AxiosResponse<Username[]>> {
    return axiosInstance.get("/users");
  }

  static async verifyCredentials(data: {
    userCredentials: UserCredentials;
  }): Promise<AxiosResponse<Username>> {
    setAuthorizationHeader(data.userCredentials);
    return axiosInstance.post("/verify-credentials");
  }

  static async getUserHistory(data: {
    userCredentials: UserCredentials | null;
  }): Promise<AxiosResponse<HistoryRecord[]>> {
    setAuthorizationHeader(data.userCredentials);
    return axiosInstance.get("/user/history");
  }

  static async createClockInRecord(data: {
    userCredentials: UserCredentials | null;
    clockInRequest: ClockInRequest;
  }): Promise<AxiosResponse<HistoryRecord>> {
    setAuthorizationHeader(data.userCredentials);
    return axiosInstance.post("/user/clock-in", data.clockInRequest);
  }

  static async createClockOutRecord(data: {
    userCredentials: UserCredentials | null;
    clockOutRequest: ClockOutRequest;
  }): Promise<AxiosResponse<HistoryRecord>> {
    setAuthorizationHeader(data.userCredentials);
    return axiosInstance.post("/user/clock-out", data.clockOutRequest);
  }
}

export default Api;

```

- We have a `models/` folder where we can put interfaces that can be shared by our API file as well as the actual components
- NOTE: type `Username` is `string` not `string[]`
    - Even though we get an array of strings of the API, it is generally easier to make a type an array than to get the type from an array
    - i.e., if `Username` is a string, it is easy to say a type is `Username[]`. However, if `Username` is `string[]` than it would be difficult to get the type of a single username
- I've made the decision to always pass in the `UserCredentials` when making a request that requires credentials to avoid the issue of keep track of when to update the `axiosInstance` with the correct headers
- I've also made the decision to allow `UserCredentials` to be null so that you can still make the request, but you would probably get a `401` error
