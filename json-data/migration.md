TODOs

```bash
POST /auth/local/register {fullName: '', email: '', password: '', retypePassword: '' }
POST /auth/local  { identifier: '', password: '' }

GET /products
GET /products/id
PATCH /products/id
POST /products


GET /categories
GET /categories/id
PATCH /categories/id
POST /categories
```


1. Fake data for /products, /categories, /login, /register


```sh
POST /auth/local
``` 

Sample Payload
```js
const payload = {
  identifier: 'abc@gmail.com', // can be either username or email
  password: '123123',
}
```

Sample Response: 
```json
{
  "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNjAzMjU0MzU0LCJleHAiOjE2MDU4NDYzNTR9.LeS0vdqri4PsSxH13nlRNiawjanoKgVNeqzR7ZQoP4B",
  "user": {
    "id": 5,
    "username": "abc@gmail.com",
    "email": "abc@gmail.com",
    "provider": "local",
    "confirmed": true,
    "blocked": null,
    "role": {
      "id": 1,
      "name": "Authenticated",
      "description": "Default role given to authenticated user.",
      "type": "authenticated",
      "created_by": null,
      "updated_by": null
    },
    "created_at": "2020-10-21T04:25:54.575Z",
    "updated_at": "2020-10-21T04:25:54.578Z",
    "created_by": null,
    "updated_by": null
  }
}
```

----

```sh
POST /auth/local/register
``` 

Sample Payload
```js
const payload = {
  email: 'abc@gmail.com',
  username: 'abc@gmail.com', // same as email
  password: '123123', // min length 6
  fullName: 'Easy Frontend',
}
```

Sample Response: 
```json
{
  "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwiaWF0IjoxNjAzMjU0MzU0LCJleHAiOjE2MDU4NDYzNTR9.LeS0vdqri4PsSxH13nlRNiawjanoKgVNeqzR7ZQoP4B",
  "user": {
    "id": 5,
    "username": "abc@gmail.com",
    "email": "abc@gmail.com",
    "provider": "local",
    "confirmed": true,
    "blocked": null,
    "role": {
      "id": 1,
      "name": "Authenticated",
      "description": "Default role given to authenticated user.",
      "type": "authenticated",
      "created_by": null,
      "updated_by": null
    },
    "created_at": "2020-10-21T04:25:54.575Z",
    "updated_at": "2020-10-21T04:25:54.578Z",
    "created_by": null,
    "updated_by": null
  }
}
```


----

Do phần API Backend của mình được dựng bằng Strapi, nên bạn có thể tham khảo cách sử dụng API ở đây nghen:

Ref: [https://strapi.io/documentation/v3.x/content-api/api-endpoints.html#endpoints](https://strapi.io/documentation/v3.x/content-api/api-endpoints.html#endpoints)

Thông thường response của một API listing, nó có thể dạng thế này: 

```js
{
  data: [],
  pagination: {
    page: 1,
    total: 12,
    limit: 10,
  }
}
```

## Một vài ví dụ truy vấn API

Lấy danh sách products, trang 1, mỗi trang 10 phần tử:

```sh
GET /products?_start=0&_limit=10
```

Lấy danh sách products, trang 3, mỗi trang 20 phần tử:

```sh
GET /products?_start=40&_limit=20
```

Như vậy, có thể nhận thấy rằng:

- `_limit`: nó là số lượng phần tử bạn muốn nhận ở mỗi trang.
- `_start`: là số lượng item bạn muốn skip (bỏ qua), vd trang 3, bỏ 2 trang đầu, tức skip 40 items, nên start=40

Nhưng tiếc là response của phần listing này không trả về thông tin pagination, nên mình phải tự làm bằng cách sử dụng một API /count để đếm xem có bao nhiêu phần tử, rồi từ đó mình sẽ tính ra được pagination.

<div style="page-break-after: always;"></div>

Ví dụ: Mình muốn lấy trang 2, mỗi trang 50 phần tử thì sẽ làm như sau: 

Gọi cả 2 APIs: `/products` và `/products/count` với parameters giống nhau.

```sh
GET /products?_start=50&_limit=50&title_contains=macbook
```

```sh
GET /products/count?_start=50&_limit=50&title_contains=macbook
```

```js
const productApi = {
  async getAll(params) {
    // Transform _page to _start
    const newParams = { ...params };
    newParams._start = !params._page || params._page <= 1
      ? 0
      : (params._page - 1) * (params._limit || 50);

    // Remove un-needed key
    delete newParams._page;

    // Fetch product list + count
    const productList = await axiosClient.get('/products', { params: newParams });
    const count = await axiosClient.get('/products/count', { params: newParams });


    // Build response and return
    return {
      data: productList,
      pagination: {
        page: params._page,
        limit: params._limit,
        total: count
      }
    }
  }
}

```