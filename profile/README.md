# Evermind - Distributed Lock as a Service

[![Evermind](https://img.shields.io/badge/Evermind-Lock%20as%20a%20Service-blue)](https://evermind.sh)
[![Polar.sh](https://img.shields.io/badge/Polar.sh-Subscribe-orange)](https://polar.sh/evermind/)
[![API Documentation](https://img.shields.io/badge/API-Documentation-green)](https://evermind.fly.dev/doc)

## 1. About Evermind

Evermind (evermind.sh) provides a simple, reliable, and scalable Distributed Lock as a Service (DLaaS), often referred to as a mutex.  We empower developers to effortlessly maintain consistency across distributed systems by offering a streamlined way to acquire, extend, and release locks on shared resources.  Leverage Evermind to simplify complex coordination challenges and build robust, fault-tolerant applications.

## 2. About the Product

Evermind's DLaaS eliminates the complexities of building and managing your own distributed locking mechanisms.  Our service offers configurable lock acquisitions with automatic retries, automatic expirations and releases, and seamless lock extensions.  Built with a serverless architecture, Evermind integrates effortlessly into any environment via its HTTP-based API.

## 3. Installation & Setup

Evermind is accessible through a simple HTTP API, making it compatible with any language or environment.  The primary endpoint is `https://lock.evermind.sh`.  Interactive Swagger documentation is available at [https://evermind.fly.dev/doc](https://evermind.fly.dev/doc).

### API Key Management

To use the lock API, you'll need an API key, managed through the `api.evermind.sh` API. You'll need a license key from your Polar.sh subscription.  You can manage your API keys easily using the Evermind CLI:

#### CLI Usage (`npx evermind keys <command>`)

Install the CLI:

```bash
npm install -g evermind
```

##### Create an API Key:

```bash
npx evermind keys create --licenceKey YOUR_POLAR_SH_LICENCE_KEY
# or set the licence key as an environment variable
export EVERMIND_LICENCE_KEY=YOUR_POLAR_SH_LICENCE_KEY
npx evermind keys create
```


##### Delete an API Key:

```bash
npx evermind keys delete YOUR_API_KEY --licenceKey YOUR_POLAR_SH_LICENCE_KEY
# or set the licence key as an environment variable
export EVERMIND_LICENCE_KEY=YOUR_POLAR_SH_LICENCE_KEY
npx evermind keys delete YOUR_API_KEY
```


#### Direct API Calls

Alternatively, you can call the API directly:

* **Create API Key:**
	* **Endpoint:** `https://api.evermind.sh/api-key`
	* **Method:** `POST`
	* **Body:** `{ "licenceKey": "YOUR_POLAR_SH_LICENCE_KEY" }`

* **Delete API Key:**
	* **Endpoint:** `https://api.evermind.sh/api-key`
	* **Method:** `DELETE`
	* **Body:** `{ "licenceKey": "YOUR_POLAR_SH_LICENCE_KEY", "apiKey": "YOUR_API_KEY" }`


Ensure you include your API key in the `Authorization` header when making requests to the `https://lock.evermind.sh` endpoint.

### Using the Lock API (TypeScript Example with SDK)

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

	// Or use a helper to manage the lifecycle of your lock
	const result = await evermind.withLock({key}, async () => {
		// Do some work, your lock will be held until this completes!

		return 'some result'
	})

	console.log("Work complete:", result);
}


run();
```

## 4. Features

* **Configurable Lock Acquisitions:**  Fine-tune lock behavior with options for lease duration, retry attempts, and retry delays.
* **Automatic Retries:**  Evermind automatically retries lock acquisitions, simplifying handling contention and transient errors.
* **Automatic Expirations and Releases:**  Locks automatically expire after a specified duration, preventing deadlocks and resource starvation.
* **Lock Extensions:** Extend the duration of a held lock without releasing it, ensuring continued exclusive access.
* **Serverless Ready:**  Built on a serverless architecture, Evermind scales effortlessly to meet demand.
* **HTTP API:**  Integrate easily with any language or environment using the simple HTTP-based API.
* **Soft Fail Option:**  Handle errors gracefully with the soft fail option, receiving error details in a 200 response instead of exceptions.
* **Convenient CLI and SDK:**  Manage API keys and interact with the lock service easily using the provided CLI and TypeScript SDK.




This updated README now includes information on using the CLI, explains how to make direct API calls, and provides a clear TypeScript example demonstrating how to use the SDK to acquire, extend, and release locks. It also adds the CLI and SDK to the features list.
