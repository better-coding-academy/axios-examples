# Axios Examples

[https://github.com/axios/axios](https://github.com/axios/axios)

# Basic API

```js
// Get request
axios
  .get('/user?ID=12345')
  .then(response => {
    // handle success
    console.log(res.data);
  })
  .catch(console.error);

// Post request
axios.post('/user', {
  firstName: 'Fred',
  lastName: 'Flintstone'
});
```

# Instances

```js
const loginService = axios.create({
  baseURL: 'https://some-domain.com/api/login/'
});

const { data } = await loginService.get('/get-token');

const customerService = axios.create({
  baseURL: 'https://some-domain.com/api/customer/',
  headers: { Authorization: data.bearerToken }
});
```

# Common

```js
// IMPORTANT: defaults are not inherited unless in axios 0.19 beta

axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;

const searchService = axios.create({
  baseURL: 'https://some-domain.com/api/search/'
});

const customerService = axios.create({
  baseURL: 'https://some-domain.com/api/customer/'
});

// Should inherit in v0.19
customerService.defaults.headers.common['Authorization'] ===
  searchService.defaults.headers.common['Authorization'];
```

# Interceptors / Transformers

```js
// Auto fetch Bearer
const instance = axios.create({ baseURL: 'https://some-domain.com/api/' });

// Intercept Request
const autoAuthorize = ({ apiKey }) => config => {
  const existing = _get(instance, 'defaults.headers.authorization');

  // Do nothing if we already have a bearer
  if (existing) return Promise.resolve(config);

  // Else fetch a new token
  return loginService.post('/get-token?key=' + apiKey).then({ data } => {
    instance.defaults.headers.authorization = data.bearerToken;
    // Replay the request with the added token
    _set(config, 'headers.authorization', data.bearerToken);
    return config;
  });
};

instance.interceptors.request.use(autoAuthorize({ apiKey: 'apple' }));

// Transform response
instance.defaults.transformResponse = (data) => xmlToJson(data, { explicitArray: false })
```

# Cancellation

```js
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios
  .get('/user/12345', {
    cancelToken: source.token
  })
  .catch(function(thrown) {
    if (axios.isCancel(thrown)) {
      console.log('Request canceled', thrown.message);
    } else {
      // handle error
    }
  });

axios.post(
  '/user/12345',
  {
    name: 'new name'
  },
  {
    cancelToken: source.token
  }
);

// cancel the request (the message parameter is optional)
source.cancel('Operation canceled by the user.');
```

# Other Useful Things

```js
// ValidateStatus
axios.post('/validate', { ...payload }, {
  validateStatus: (status) => status >= 200 && status < 400;
});

// Https agent
// Ignore self signed ssl
const internalAgent = new https.Agent({ rejectUnauthorized: false });

axios
  .post(
    CSG_TOKEN_URL,
    { ...payload },
    {
      httpsAgent: NODE_ENV !== 'production' && internalAgent
    }
  )
```