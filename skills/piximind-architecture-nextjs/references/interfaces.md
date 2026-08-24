# Route, response, and DS interfaces

Inspect the repo’s existing interface folder and router const. Copy that layout. Do not add a second contracts tree.

## Where files go

| Kind | Location (pick the one already in the repo) | Name |
|------|-----------------------------------------------|------|
| Backend / BFF **paths** | Existing router const (`BACKEND_API_ROUTER`, `FRONTEND_API_ROUTER`, or `*_END_POINT`) | Nested object keys. No raw path strings in `.api.ts` or `route.ts`. |
| Nest / fetch **responses** | `{domain}.api.interface.ts` next to `{domain}.api.ts`, **or** `common-side/interface/api/`, **or** `common/interface/services/api/` / `common/interface/{domain}/` | `I{Domain}{Action}Response` / `I{Domain}ListApiResponse` |
| Shared HTTP envelope | Same api-interface folder | `ApiResponse<T>` — `data: T`, never `data: any` |
| JWT / session | Same api-interface folder (server-safe module) | `IDecodedToken`, `ILoginResponse`, `ISessionData` |
| DS **props** | See `piximind-atomic-web` | `IAtom*`, `IMolecule*`, `IOrganism*` |

Do not mix colocated `{domain}.api.interface.ts` with a new `common/interface/services/api/` in the same repo.

## Route Handler contract

```ts
export async function GET(req: Request): Promise<NextResponse> {
  const apiRes: IUsersListResponse = await UsersApi.getInstance().list(accessToken, query);
  const body: IUsersTableResponse = mapUsersList(apiRes);
  return NextResponse.json(body, { status: 200 });
}
```

- Annotate the Nest/BFF result with the response interface. No inline `{ list?: ... }` casts.
- Mapping belongs in `*.mapper.ts`, not a 150-line `route.ts`.
- Paths only via the router const.

## API module contract

```ts
export interface IUsersListResponse {
  data: { list: IUserRow[]; total: number; numberOfPages: number };
  status: number;
}

class UsersApi {
  public async list(accessToken: string, query: IUsersListQuery): Promise<IUsersListResponse> {
    return this._api.call({
      type: EApiType.get,
      endPoint: `${BACKEND_API_ROUTER.users.list}?${query}`,
      headers,
    });
  }
}
```

Exported methods always declare the `Promise<…>` return type. Do not return `Promise<any>` or untyped `ApiResponse`.

## JWT

```ts
const decoded: IDecodedToken = jwtDecode<IDecodedToken>(accessToken);
```

`IDecodedToken` lists known claims (`exp`, `sub`, `email`, `resource_access`, …). Never `jwtDecode(token)` untyped, never `as any` on the payload. Do not put refresh tokens or secrets on types imported by client DS.

## Forbidden

- `any`, `as any`, `Record<string, any>` — use generics or `unknown` + narrow.
- Duplicated `"/admin/users"` strings beside the router const.
- Response shapes declared only inside `route.ts`, a page, or an atom.
- Secrets, raw env, or Bearer tokens on interfaces under `client-side/ds`.
