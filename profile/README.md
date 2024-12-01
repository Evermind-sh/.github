# Evermind - Distributed Lock as a Service

![Evermind](https://img.shields.io/badge/Evermind-Lock%20as%20a%20Service-blue)
![Polar.sh](https://img.shields.io/badge/Polar.sh-Subscribe-orange)
![API Documentation](https://img.shields.io/badge/API-Documentation-green)

---

## 1. About Evermind

[Evermind](https://evermind.sh) is a simple, reliable, and scalable Distributed Lock as a Service (DLaaS), also referred to as a mutex. It empowers developers to maintain consistency across distributed systems effortlessly by offering tools to acquire, extend, and release locks on shared resources. Evermind simplifies complex coordination challenges, enabling robust, fault-tolerant applications.

---

## 2. About the Product

Evermind eliminates the complexities of building and managing distributed locking mechanisms. Our service provides:

- Configurable lock acquisitions with automatic retries.
- Automatic expirations and releases.
- Seamless lock extensions.
- A serverless-ready architecture that integrates easily into any environment.

Subscribe to a plan via our storefront: [Polar.sh Evermind](https://polar.sh/evermind/).

---

## 3. Setup: Acquiring an API Key

To interact with Evermind's lock API, you first need to exchange your Polar.sh license key for an API key. API keys are managed via the `https://api.evermind.sh` service.

### 3.1 Using the CLI

Install the CLI globally:

```bash
npm install -g evermind
```

**Create an API key:**

```bash
npx evermind keys create --licenceKey YOUR_POLAR_SH_LICENCE_KEY
```

Or use an environment variable:

```bash
export EVERMIND_LICENCE_KEY=YOUR_POLAR_SH_LICENCE_KEY
npx evermind keys create
```

**Delete an API key:**

```bash
npx evermind keys delete YOUR_API_KEY --licenceKey YOUR_POLAR_SH_LICENCE_KEY
```

Or use an environment variable:

```bash
export EVERMIND_LICENCE_KEY=YOUR_POLAR_SH_LICENCE_KEY
npx evermind keys delete YOUR_API_KEY
```

---

### 3.2 Direct HTTP API Calls

#### Create API Key

- **Endpoint:** `https://api.evermind.sh/api-key`
- **Method:** `POST`
- **Body:**
  ```json
  {
    "licenceKey": "YOUR_POLAR_SH_LICENCE_KEY"
  }
  ```
- **Response:**
  ```json
  {
    "apiKey": "YOUR_API_KEY"
  }
  ```

#### Delete API Key

- **Endpoint:** `https://api.evermind.sh/api-key`
- **Method:** `DELETE`
- **Body:**
  ```json
  {
    "licenceKey": "YOUR_POLAR_SH_LICENCE_KEY",
    "apiKey": "YOUR_API_KEY"
  }
  ```
- **Response:**
  204 No Content (if successful)

---

## 4. Using Evermind

Once you have an API key, you can interact with the lock service to acquire, extend, and release locks. The following sections describe how to use the system directly via HTTP methods and with the TypeScript SDK.

### 4.1 TypeScript SDK Usage

Install the SDK:

```bash
npm install evermind
```

**Example Usage:**

```typescript
import { Evermind } from 'evermind';

const evermind = new Evermind({ apiKey: 'YOUR_API_KEY' });

async function run() {
    const key = 'my_resource';

    // Acquire a lock
    const acquireResponse = await evermind.acquire({ key, lease: 5000 });
    console.log('Acquire:', acquireResponse);

    // Extend a lock
    if (acquireResponse.acquired) {
        const extendResponse = await evermind.extend({ key, uuid: acquireResponse.config.uuid, extendBy: 2000 });
        console.log('Extend:', extendResponse);

        // Release the lock
        const releaseResponse = await evermind.release({ key, uuid: acquireResponse.config.uuid, softFail: true });
        console.log('Release:', releaseResponse);
    }

    // Using `withLock` to simplify lock management
    const result = await evermind.withLock({ key }, async () => {
        // Do some work while holding the lock
        return 'some result';
    });

    console.log("Work complete:", result);
}

run();
```

---

### 4.2 HTTP API Examples

If preferred (or are not using TypeScript/Node.js) then you can call each of the endpoints directly using any HTTP client.

#### Acquire Lock

- **Endpoint:** `https://lock.evermind.sh/lock/acquire`
- **Method:** `POST`
- **Body:**
  ```json
  {
    "key": "resource_key",
    "lease": 5000,
    "retryAttempts": 5,
    "retryDelay": 500,
    "uuid": "custom-uuid",
    "softFail": false
  }
  ```
- **Response:**
  ```json
  {
    "acquired": true,
    "lockAcquisitionAttempts": 1,
    "config": {
      "key": "resource_key",
      "lease": 5000,
      "retryAttempts": 5,
      "retryDelay": 500,
      "uuid": "custom-uuid"
    },
    "message": null
  }
  ```

#### Extend Lock

- **Endpoint:** `https://lock.evermind.sh/lock/extend`
- **Method:** `POST`
- **Body:**
  ```json
  {
    "key": "resource_key",
    "uuid": "custom-uuid",
    "extendBy": 3000,
    "softFail": true
  }
  ```
- **Response:**
  ```json
  {
    "extended": true,
    "message": null
  }
  ```

#### Release Lock

- **Endpoint:** `https://lock.evermind.sh/lock/release`
- **Method:** `POST`
- **Body:**
  ```json
  {
    "key": "resource_key",
    "uuid": "custom-uuid",
    "softFail": true
  }
  ```
- **Response:**
  ```json
  {
    "released": true,
    "message": null
  }
  ```

---

## 5. DTOs

### Lock API DTOs

#### Acquire Lock

| **Field**       | **Type**    | **Required** | **Description**                                         |
|------------------|-------------|--------------|---------------------------------------------------------|
| `key`           | `string`    | Yes          | The resource to lock.                                   |
| `lease`         | `number`    | No           | Lock duration in milliseconds. Default: `10000`.       |
| `retryAttempts` | `number`    | No           | Number of retry attempts. Default: `10`.               |
| `retryDelay`    | `number`    | No           | Milliseconds between retries. Default: `500`.          |
| `uuid`          | `string`    | No           | Optional custom UUID for the lock.                     |
| `softFail`      | `boolean`   | No           | Return errors inline (`200`) instead of exceptions.     |

#### Extend Lock

| **Field**       | **Type**    | **Required** | **Description**                                         |
|------------------|-------------|--------------|---------------------------------------------------------|
| `key`           | `string`    | Yes          | The resource key to extend the lock on.                |
| `uuid`          | `string`    | Yes          | UUID of the lock instance to extend.                   |
| `extendBy`      | `number`    | Yes          | Milliseconds to extend the lock.                       |
| `softFail`      | `boolean`   | No           | Return errors inline (`200`) instead of exceptions.     |

#### Release Lock

| **Field**       | **Type**    | **Required** | **Description**                                         |
|------------------|-------------|--------------|---------------------------------------------------------|
| `key`           | `string`    | Yes          | The resource key to release the lock on.               |
| `uuid`          | `string`    | Yes          | UUID of the lock instance to release.                  |
| `softFail`      | `boolean`   | No           | Return errors inline (`200`) instead of exceptions.     |

---

## 6. Features

- **Configurable Lock Acquisitions:** Fine-tune lock behavior with options for lease duration, retries, and delay intervals.
- **Automatic Expirations and Releases:** Prevent deadlocks and resource starvation.
- **Lock Extensions:** Extend a lock without releasing it.
- **HTTP API & TypeScript SDK:** Flexible integrations with multiple environments.
- **Soft Fail Option:** Handle errors gracefully with inline responses.

---

## 7. Discussion & Alternatives

Evermind offers a managed DLaaS solution. However, various alternatives exist for concurrency control. Here's a comparison to help you choose the right tool:

### Distributed Locks vs. Local Mutexes

- **Local Mutexes:** Efficient for single-process applications. Libraries like `sync.Mutex` (Go), `asyncio.Lock` (Python), and `@synchronized` (Objective-C/Swift) operate within a single process's memory.
- **Distributed Locks:** Necessary for coordinating shared resources across multiple processes or machines. This is where Evermind excels.

### Alternatives to Evermind

- **Redlock Algorithm (e.g., npm package `redlock`):** Fault-tolerant distributed locking using multiple Redis instances. However, managing Redis infrastructure introduces additional complexity.
- **Database-based Locking:** Simple to implement but may cause performance bottlenecks in high-contention scenarios.
- **Distributed Consensus Systems (e.g., etcd, ZooKeeper):** Offer robust locking with higher operational overhead.

### When to Use Evermind

- Simplifying distributed locking without managing infrastructure.
- Seamless integration with serverless environments.
- Rapid development with minimal setup.

### When Not to Use Evermind

- Single-process applications (use local mutexes instead).
- Applications already using robust Redis infrastructure and comfortable with Redlock.
- Extremely low-latency scenarios that require finely-tuned, in-house solutions.

---
