# **A Definitive Guide to Integrating and Securing Bitrequest Payments in Next.js and Medusa.js**

## **Section 1: Deconstructing the Bitrequest Payment Flow**

Implementing a robust cryptocurrency payment solution requires a foundational understanding of its architecture and data flow. The Bitrequest platform presents a unique model that empowers merchants by being non-custodial while necessitating a specific and security-conscious integration strategy. This section deconstructs the core components of a Bitrequest transaction, from its architectural principles to the granular structure of its communication protocols, establishing the necessary context for a secure e-commerce implementation.

### **1.1 Architectural Overview: The Non-Custodial, Client-Side Approach**

Bitrequest is designed as a "Non-custodial, client-side PWA (Progressive Web App) with real-time monitoring". This architectural choice has profound implications for the merchant. "Non-custodial" signifies that the platform never takes possession of the merchant's funds or private keys. All transactions occur directly between the customer and the merchant's wallet, granting the merchant complete control and sovereignty over their assets. The "client-side" nature means that the primary application logic, including the payment interface and transaction monitoring, executes within the end-user's browser.  
This architecture supports three primary use cases: a physical Point of Sale (POS) system, direct peer-to-peer payment requests via URL sharing, and, most relevant to this guide, integration into e-commerce webshops for checkout flows.  
A critical feature of this model is its method for providing "instant payment feedback." Bitrequest achieves this not through a proprietary, centralized ledger but by actively monitoring public blockchain data sources via "WebSocket/polling on public explorers/nodes". When a transaction corresponding to a payment request is detected on the network, the client-side application receives a notification and relays it to the merchant's website. This mechanism provides a responsive user experience but, as will be explored, places the burden of definitive verification squarely on the merchant's backend system. The client-side notification is an optimistic signal of a detected payment, not a cryptographically secure confirmation of its validity or finality. This distinction is the cornerstone of a secure integration.

### **1.2 The Anatomy of a Payment Request URL**

The entire Bitrequest payment process is initiated through a specially crafted URL. This URL is not merely a link but a complete, self-contained instruction set that tells the Bitrequest PWA what to charge, in what currency, and where to send the funds. Understanding its structure is the first step in dynamically generating payment requests from an e-commerce backend.  
The base URL structure is as follows: https://bitrequest.github.io/?payment=\<crypto\>\&uoa=\<fiat\>\&amount=\<value\>\&address=\<your\_address\>\&d=\<data\>  
Each query parameter serves a distinct purpose in defining the transaction, as detailed in the platform's integration examples.

| Parameter | Type | Required | Description | Example Value |
| :---- | :---- | :---- | :---- | :---- |
| payment | String | Yes | The cryptocurrency the customer will pay with. Supported options include Bitcoin, Lightning, Nano, Ethereum, and many others. | bitcoin |
| uoa | String | Yes | The Unit of Account, typically a fiat currency, used to price the goods. This enables fiat-pegged pricing. | usd |
| amount | Number | Yes | The price of the goods in the specified uoa. | 99.95 |
| address | String | Yes | The merchant's public receiving address for the specified payment currency. | bc1q... |
| d | String | Yes | A Base64-encoded JSON string containing metadata about the payment, most importantly a unique payment identifier. | eyJwaWQiOiJ... |
| contactform | Boolean | No | An optional flag that, when set to true, prompts the user for shipping details within the Bitrequest UI. | true |

This parameterized URL design allows for a stateless initiation of the payment process. The merchant's server generates this URL, the frontend displays it, and the Bitrequest PWA takes over from there, possessing all the information it needs to manage the transaction with the customer.

### **1.3 The Base64 Encoded Data (d) Parameter: Linking Payments to Orders**

While other URL parameters define the financial aspects of the transaction, the d parameter is the critical data bridge that links the anonymous cryptocurrency payment back to a specific order within the e-commerce system. It is a Base64-encoded JSON object that the merchant's backend constructs.  
A typical structure for this JSON object is:  
`{`  
  `"t": "Order #12345 from My Webshop",`  
  `"n": "Payment for T-Shirt and Mug",`  
  `"c": 0,`  
  `"pid": "pay_session_01H8X2J4Z5N6D8P7Q9R1T3V5F7"`  
`}`

The encoding process in JavaScript would be btoa(JSON.stringify(dataObject)).  
While fields like t (title) and n (name) are useful for display purposes within the Bitrequest interface, the pid (payment ID) field is of paramount importance for the backend. This pid **must** be a unique, non-guessable, and cryptographically secure identifier generated by the merchant's server. In the context of a Medusa.js integration, the ID of the PaymentSession is an ideal candidate for this value. This identifier serves as the primary key for retrieving the correct order details when Bitrequest sends a payment notification. It is the sole mechanism for preventing transaction ambiguity and is fundamental to ensuring idempotency in the payment processing logic.

### **1.4 The postMessage Callback: Receiving Payment Notifications**

Once a customer completes a payment and Bitrequest detects the transaction on the corresponding blockchain network, it communicates this event back to the merchant's website. This communication does not happen via a server-to-server webhook. Instead, it uses the browser's standard window.postMessage API. The Bitrequest PWA, running in an iframe, sends a message to the parent window (the e-commerce site).  
To receive this message, the merchant's frontend code must define a global JavaScript function named result\_callback. This function acts as the designated event listener for all payment status updates from Bitrequest.  
`function result_callback(post_data) {`  
  `console.log("Received payment notification:", post_data);`  
  `// Logic to handle the notification will be implemented here.`  
`}`

The post\_data object passed to this function contains a wealth of information, structured into two key sub-objects.

| Key Path | Type | Description | Example |
| :---- | :---- | :---- | :---- |
| id | String | The type of message being sent. For payment results, this is typically "result". | "result" |
| data.txdata.status | String | The status of the payment as detected by Bitrequest. A value of "paid" indicates a transaction has been seen on the network. | "paid" |
| data.txdata.txhash | String | The transaction hash (ID) of the payment on its respective blockchain. This is **critical** for server-side verification. | "a1b2c3..." |
| data.txdata.receivedamount | String | The amount of cryptocurrency received, as a string. | "0.0015" |
| data.txdata.receiver | String | The receiving address the funds were sent to. | "bc1q..." |
| data.data.pid | String | The original Payment ID that was passed in the d parameter of the request URL. This is the key to linking the payment to an order. | "pay\_session\_..." |

The txdata object contains the details of the on-chain event as observed by Bitrequest's monitoring system. The data object contains the decoded payload that was originally sent in the d parameter, providing the essential pid. The arrival of this post\_data object at the result\_callback function is the trigger that initiates the next phase of the process: securely verifying the payment and fulfilling the order. The data within this object—specifically the txhash, receiver, receivedamount, and pid—should be treated as the *parameters* for a mandatory server-side verification process, not as a final, trustworthy confirmation of payment itself.

## **Section 2: Frontend Implementation for Next.js and Medusa.js Storefronts**

The frontend of a Bitrequest integration is responsible for two primary tasks: dynamically rendering the payment button with the correct, backend-generated URL, and securely capturing the payment notification to relay it for server-side processing. This section provides a complete guide to implementing these components within a modern Next.js storefront that communicates with a Medusa.js backend.

### **2.1 Initial Setup: Incorporating the Bitrequest Checkout Library**

To enable the Bitrequest payment modal, two external resources must be included in the application: a stylesheet for styling and a JavaScript library that contains the logic for hijacking links and opening the payment iframe. In a Next.js application, these are best managed using the built-in Head and Script components for optimal loading and performance.  
These resources should be added to the main layout file (e.g., app/layout.tsx in the App Router) or a specific checkout page component to ensure they are loaded when needed.  
`// app/layout.tsx (or a more specific component)`  
`import Script from 'next/script';`

`export default function RootLayout({ children }: { children: React.ReactNode }) {`  
  `return (`  
    `<html lang="en">`  
      `<head>`  
        `<link`  
          `rel="stylesheet"`  
          `href="https://bitrequest.github.io/assets_styles_lib_bitrequest.css"`  
        `/>`  
      `</head>`  
      `<body>`  
        `{children}`  
        `<Script`  
          `src="https://bitrequest.github.io/assets_js_lib_bitrequest_checkout.js"`  
          `strategy="lazyOnload" // Load the script after the page is interactive`  
        `/>`  
      `</body>`  
    `</html>`  
  `);`  
`}`

By including these files, any anchor tag (\<a\>) with the class name br\_checkout will automatically have its click event intercepted by the Bitrequest script, which will then open the payment modal using the URL provided in the href attribute.

### **2.2 Building a Dynamic Checkout Component in Next.js**

The payment button cannot use a static URL. It must be dynamically generated for each unique cart and payment session, containing the correct amount, currency, and, most importantly, the unique payment ID (pid). This requires a React component that receives payment session data from the Medusa.js backend and constructs the URL accordingly.  
The Medusa.js backend, as will be detailed in Section 3, will prepare all the necessary URL parameters and store them in the payment\_session.data object. The frontend component's job is to read this data and render the button.  
Here is an example of a BitrequestButton.tsx component:  
`// components/checkout/BitrequestButton.tsx`  
`'use client';`

`import { PaymentSession } from '@medusajs/medusa';`  
`import { useMemo } from 'react';`

`type BitrequestButtonProps = {`  
  `session: PaymentSession;`  
`};`

`// Helper function to build the URL from session data`  
`const buildRequestUrl = (data: Record<string, any>): string | null => {`  
  `const { payment, uoa, amount, address, d } = data;`  
  `if (!payment ||!uoa ||!amount ||!address ||!d) {`  
    `console.error("Bitrequest session data is incomplete.");`  
    `return null;`  
  `}`  
  ``return `https://bitrequest.github.io/?payment=${payment}&uoa=${uoa}&amount=${amount}&address=${address}&d=${d}`;``  
`};`

`const BitrequestButton: React.FC<BitrequestButtonProps> = ({ session }) => {`  
  `const requestUrl = useMemo(() => buildRequestUrl(session.data), [session.data]);`

  `if (!requestUrl) {`  
    `return (`  
      `<div className="text-red-500">`  
        `Error: Unable to initialize Bitrequest payment. Please try again.`  
      `</div>`  
    `);`  
  `}`

  `return (`  
    `<a`  
      `href={requestUrl}`  
      `className="br_checkout w-full bg-blue-600 text-white py-3 px-6 rounded-md text-center font-semibold hover:bg-blue-700 transition-colors"`  
    `>`  
      `Pay with Cryptocurrency`  
    `</a>`  
  `);`  
`};`

`export default BitrequestButton;`

This component receives the Medusa.js PaymentSession object as a prop. It uses useMemo to construct the request\_url from the session.data field, ensuring the URL is only recalculated when the session data changes. It then renders the required anchor tag with the br\_checkout class, ready for the Bitrequest library to activate it.

### **2.3 Implementing the result\_callback Handler for State Management**

The global result\_callback function is the entry point for all payment notifications from the Bitrequest iframe. In a React/Next.js application, this function must be carefully attached to the global window object and should be designed to interact with the application's state and backend services.  
A common and effective pattern is to define this handler within a checkout context or a high-level component that manages the checkout flow. The useEffect hook is perfect for attaching the function to the window object when the component mounts and cleaning it up when it unmounts.  
This handler's logic should be minimal and focused: parse the incoming data, trigger a backend verification call, and update the UI to reflect a "verifying" state.  
`// components/checkout/CheckoutFlow.tsx`  
`'use client';`

`import { useEffect, useState } from 'react';`  
`import { useMutation } from '@tanstack/react-query';`  
`import axios from 'axios';`  
`import { useCart } from 'medusa-react'; // Assuming usage of medusa-react`

`// Define the structure of the callback data for type safety`  
`interface BitrequestCallbackData {`  
  `id: string;`  
  `data: {`  
    `txdata: any; // Define more specific types as needed`  
    `data: {`  
      `pid: string;`  
    `};`  
  `};`  
`}`

`// Define the payload for our backend verification API`  
`interface VerificationPayload {`  
  `cart_id: string;`  
  `payment_session_id: string;`  
  `tx_data: any;`  
`}`

`const verifyPayment = async (payload: VerificationPayload) => {`  
  `const { data } = await axios.post('/api/store/bitrequest/callback', payload);`  
  `return data;`  
`};`

`const CheckoutFlow = () => {`  
  `const { cart } = useCart();`  
  `const = useState<'idle' | 'verifying' | 'success' | 'error'>('idle');`

  `const mutation = useMutation({`  
    `mutationFn: verifyPayment,`  
    `onSuccess: () => {`  
      `setPaymentStatus('success');`  
      `// Here you would typically redirect to an order confirmation page`  
      `// The cart should be completed by the backend, so a redirect is safe`  
      ``window.location.href = `/order/confirmed/${cart?.id}`;``  
    `},`  
    `onError: (error) => {`  
      `console.error("Payment verification failed:", error);`  
      `setPaymentStatus('error');`  
    `},`  
  `});`

  `useEffect(() => {`  
    `const handlePaymentResult = (postData: BitrequestCallbackData) => {`  
      `console.log('Bitrequest callback received:', postData);`

      `if (postData?.id === 'result' && postData?.data?.data?.pid && cart?.id) {`  
        `setPaymentStatus('verifying');`

        `mutation.mutate({`  
          `cart_id: cart.id,`  
          `payment_session_id: postData.data.data.pid,`  
          `tx_data: postData.data.txdata,`  
        `});`  
      `}`  
    `};`

    `// Attach the function to the window object`  
    `(window as any).result_callback = handlePaymentResult;`

    `// Cleanup function to remove it when the component unmounts`  
    `return () => {`  
      `delete (window as any).result_callback;`  
    `};`  
  `}, [cart?.id, mutation]);`

  `// Render UI based on paymentStatus`  
  `if (paymentStatus === 'verifying') {`  
    `return <div>Verifying your payment, please wait...</div>;`  
  `}`  
  `if (paymentStatus === 'error') {`  
    `return <div>There was an error verifying your payment. Please contact support.</div>;`  
  `}`

  `//... render the rest of the checkout form`  
  `return (`  
    `<div>`  
      `{/*... other checkout components... */}`  
      `{cart?.payment_session?.provider_id === 'bitrequest' && (`  
        `<BitrequestButton session={cart.payment_session} />`  
      `)}`  
    `</div>`  
  `);`  
`};`

### **2.4 Securely Forwarding Payment Data to a Backend API Route**

The final step on the frontend is to create the API endpoint that the result\_callback handler calls. This endpoint acts as a secure proxy, forwarding the untrusted client-side data to the trusted Medusa.js backend environment. Using a Next.js API route for this purpose is ideal as it runs on the server and can securely manage communication with the Medusa backend, which might be hosted on a different domain.  
This route serves as a critical security boundary. It translates the insecure, browser-based postMessage event into a secure, server-to-server API call.  
`// app/api/store/bitrequest/callback/route.ts`  
`import { NextRequest, NextResponse } from 'next/server';`

`// The URL of your Medusa.js backend`  
`const MEDUSA_BACKEND_URL = process.env.NEXT_PUBLIC_MEDUSA_BACKEND_URL |`

`| 'http://localhost:9000';`

`export async function POST(req: NextRequest) {`  
  `try {`  
    `const body = await req.json();`  
    `const { cart_id, payment_session_id, tx_data } = body;`

    `if (!cart_id ||!payment_session_id ||!tx_data) {`  
      `return NextResponse.json({ message: 'Missing required parameters' }, { status: 400 });`  
    `}`

    `// Forward the data to the Medusa backend's custom endpoint`  
    `// This call should be authenticated if your Medusa API is protected`  
    ``const medusaResponse = await fetch(`${MEDUSA_BACKEND_URL}/store/bitrequest/callback`, {``  
      `method: 'POST',`  
      `headers: {`  
        `'Content-Type': 'application/json',`  
        `// Include any necessary authentication headers, e.g., session cookies`  
      `},`  
      `body: JSON.stringify({`  
        `cart_id,`  
        `payment_session_id,`  
        `tx_data,`  
      `}),`  
    `});`

    `if (!medusaResponse.ok) {`  
      `const errorData = await medusaResponse.json();`  
      `throw new Error(errorData.message |`

`| 'Medusa backend verification failed');`  
    `}`

    `const responseData = await medusaResponse.json();`

    `// The Medusa backend will complete the cart and create the order.`  
    `// The success response here signals the frontend to redirect.`  
    `return NextResponse.json(responseData, { status: 200 });`

  `} catch (error: any) {`  
    `console.error('Error in Bitrequest callback proxy:', error);`  
    `return NextResponse.json({ message: error.message |`

`| 'Internal Server Error' }, { status: 500 });`  
  `}`  
`}`

With this frontend architecture in place, the client-side responsibilities are fulfilled. The system is prepared to display a payment option, listen for the result, and securely delegate the critical task of verification to the backend, ensuring no fulfillment decisions are ever made based on untrusted data.

## **Section 3: Backend Integration: Building a Medusa.js Payment Processor**

The heart of a secure Bitrequest integration lies in the backend. Within the Medusa.js ecosystem, this is accomplished by creating a custom Payment Processor. This service translates the asynchronous, callback-driven nature of Bitrequest into the structured, stateful transaction flow that Medusa's core commerce engine expects. This section details the step-by-step process of building this essential service, adapting Medusa's standard payment lifecycle to accommodate Bitrequest's unique model.

### **3.1 Scaffolding the Bitrequest Payment Processor Service**

A Medusa.js Payment Processor is a service class that extends the AbstractPaymentProcessor. The first step is to create the file and class structure. The file should be placed in src/services/ and named according to the class name (e.g., BitrequestPaymentService becomes bitrequest-payment.ts).  
The class must have a static identifier property. This string is how the processor is identified throughout the Medusa application, both in the database and when enabling it for a region in the admin dashboard.  
`// src/services/bitrequest-payment.ts`  
`import {`  
  `AbstractPaymentProcessor,`  
  `PaymentProcessorContext,`  
  `PaymentProcessorError,`  
  `PaymentProcessorSessionResponse,`  
  `PaymentSessionStatus,`  
`} from "@medusajs/medusa";`

`class BitrequestPaymentService extends AbstractPaymentProcessor {`  
  `static identifier = "bitrequest";`

  `constructor(container, options) {`  
    `super(container);`  
    `// Options can be used to configure addresses, etc. from medusa-config.js`  
    `this.options = options;`  
  `}`

  `// Methods will be implemented in the following sections`  
`}`

`export default BitrequestPaymentService;`

### **3.2 initiatePayment: Generating the Request URL Parameters**

The initiatePayment method is called by Medusa when a customer selects Bitrequest as their payment method and proceeds in the checkout flow. Its primary responsibility is not to process a payment but to prepare all the necessary data for the frontend to render the payment button.  
In this method, the backend will construct the components of the Bitrequest URL. It will generate the unique payment ID (pid), create the JSON data object, Base64 encode it, and store all these pieces in the data property of the PaymentSession. This data object is then automatically sent to the storefront as part of the cart object, where the frontend component from Section 2 can access it.  
`// Inside BitrequestPaymentService class`

`async initiatePayment(`  
  `context: PaymentProcessorContext`  
`): Promise<PaymentProcessorError | PaymentProcessorSessionResponse> {`  
  `const { currency_code, amount, resource_id, payment_session } = context;`

  `// For this example, we'll handle Bitcoin payments.`  
  `// A production implementation would need logic to select the currency`  
  `// and corresponding address based on user choice or configuration.`  
  `const paymentType = 'bitcoin'; // This could be dynamic`  
  `const merchantAddress = this.options.bitcoinAddress; // Configured in medusa-config.js`

  `if (!merchantAddress) {`  
    `return this.buildError("Configuration error: Bitcoin address not set for Bitrequest.", "invalid_argument");`  
  `}`

  `const paymentId = payment_session.id; // Use the payment session ID as the unique PID`

  `const dataPayload = {`  
    `pid: paymentId,`  
    ``t: `Order for cart ${resource_id}`,``  
    ``n: `Total: ${(amount / 100).toFixed(2)} ${currency_code.toUpperCase()}`,``  
    `c: 0,`  
  `};`

  `const encodedData = Buffer.from(JSON.stringify(dataPayload)).toString('base64');`

  `const session_data = {`  
    `payment: paymentType,`  
    `uoa: currency_code.toLowerCase(),`  
    `amount: (amount / 100).toFixed(2), // Convert from cents/minor units`  
    `address: merchantAddress,`  
    `d: encodedData,`  
  `};`

  `return {`  
    `session_data,`  
  `};`  
`}`

### **3.3 Creating a Custom API Endpoint for Callback Data Ingestion**

The standard AbstractPaymentProcessor interface does not include a method for handling asynchronous, client-initiated callbacks. Therefore, a custom API endpoint must be created within Medusa to serve as the secure entry point for notifications forwarded by the Next.js frontend.  
This endpoint will receive the tx\_data and payment\_session\_id (pid), retrieve the relevant payment session, and then trigger the crucial server-side verification logic (detailed in Section 4). If verification is successful, this endpoint will be responsible for completing the cart, which internally calls the authorizePayment method.  
`// src/api/store/bitrequest/callback/route.ts`  
`import { MedusaRequest, MedusaResponse } from "@medusajs/medusa";`  
`import { CartService } from "@medusajs/medusa";`  
`import VerificationService from "../../../../services/verification"; // The service from Section 4`

`export async function POST(req: MedusaRequest, res: MedusaResponse) {`  
  `const { cart_id, payment_session_id, tx_data } = req.body;`

  `if (!cart_id ||!payment_session_id ||!tx_data) {`  
    `return res.status(400).json({ message: "Missing required parameters" });`  
  `}`

  `try {`  
    `const cartService: CartService = req.scope.resolve("cartService");`  
    `const verificationService: VerificationService = req.scope.resolve("verificationService");`

    `const cart = await cartService.retrieve(cart_id, { relations: ["payment_session"] });`

    `// Idempotency Check: Ensure the order isn't already created`  
    `if (cart.completed_at) {`  
      `return res.status(200).json({ message: "Order already completed." });`  
    `}`

    `if (cart.payment_session?.id!== payment_session_id) {`  
      `return res.status(400).json({ message: "Payment session mismatch." });`  
    `}`

    `// --- Server-Side Verification ---`  
    `const isVerified = await verificationService.verifyTransaction(tx_data, cart.payment_session);`

    `if (!isVerified) {`  
      `return res.status(400).json({ message: "Payment verification failed." });`  
    `}`

    `// If verified, complete the cart (which creates the order)`  
    `const { data: order } = await cartService.complete(cart_id);`

    `// Store the verified transaction hash on the payment object for records`  
    `const paymentService = req.scope.resolve("paymentService");`  
    `await paymentService.update(order.payments.id, {`  
      `metadata: { tx_hash: tx_data.txhash },`  
    `});`

    `return res.status(200).json({ order });`

  `} catch (error) {`  
    `console.error("Error in Medusa Bitrequest callback:", error);`  
    `return res.status(500).json({ message: "Internal server error during payment processing." });`  
  `}`  
`}`

### **3.4 authorizePayment and Managing Payment State**

In a standard payment flow, authorizePayment is called to get approval from a third-party provider before an order is placed. In our adapted model, this method's role is simplified. It is called internally by cartService.complete() *after* our custom endpoint has already performed the definitive verification.  
Its job is to confirm the payment session's status and store any final, verified data from the transaction onto the Payment object that Medusa is creating. The data property of the Payment model is the ideal place to store the verified transaction hash and other on-chain details for auditing and record-keeping.  
`// Inside BitrequestPaymentService class`

`async authorizePayment(`  
  `paymentSession: PaymentSession,`  
  `context: Record<string, unknown>`  
`): Promise<PaymentProcessorError | {`  
  `status: PaymentSessionStatus;`  
  `data: Record<string, unknown>;`  
`}> {`  
  `// At this point, our custom endpoint has already verified the payment.`  
  `// This method's role is to formalize the authorization within Medusa's state machine.`

  `return {`  
    `status: PaymentSessionStatus.AUTHORIZED,`  
    `data: {`  
      `// We can pass the verified data from the context if needed,`  
      `// but our custom endpoint already updated the payment metadata.`  
     `...paymentSession.data,`  
    `},`  
  `};`  
`}`

`async getPaymentStatus(`  
  `paymentSessionData: Record<string, unknown>`  
`): Promise<PaymentSessionStatus> {`  
  `// This method is called by Medusa to check the status.`  
  `// For Bitrequest, the status is effectively 'pending' until our callback verifies it.`  
  `// After verification, the session is authorized and this method is less critical.`  
  `return PaymentSessionStatus.PENDING;`  
`}`

### **3.5 Implementing capturePayment, refundPayment, and cancelPayment**

The remaining methods of the AbstractPaymentProcessor must be implemented to handle the full order lifecycle.

* **capturePayment**: For cryptocurrencies, the payment is effectively "captured" the moment it is confirmed on the blockchain. Our verification process confirms this. Therefore, the capturePayment method in Medusa, typically triggered manually by an admin, will simply return the existing payment data, confirming it is already secured.  
* **refundPayment**: Refunding a cryptocurrency transaction cannot be automated through Bitrequest or a simple API call. It is an out-of-band process that requires the merchant to manually create and send a new transaction from their wallet to the customer. This method should therefore return an error or update the order's status to "refund requested," signaling the need for manual intervention.  
* **cancelPayment**: This method is used when an order is canceled before capture. For Bitrequest, it simply involves updating the state within Medusa. It has no on-chain effect, as the original payment request remains valid until it expires or is paid.

`// Inside BitrequestPaymentService class`

`async capturePayment(`  
  `payment: Payment`  
`): Promise<PaymentProcessorError | Record<string, unknown>> {`  
  `// The payment was already verified and "captured" on-chain.`  
  `// This method confirms that state to Medusa.`  
  `return payment.data;`  
`}`

`async refundPayment(`  
  `payment: Payment,`  
  `refundAmount: number`  
`): Promise<PaymentProcessorError | Record<string, unknown>> {`  
  `// This is a manual process. Log the request and alert the admin.`  
  ``console.log(`Manual refund requested for ${refundAmount} for payment ${payment.id}`);``  
  `// You can add logic here to send an email or notification.`  
  `// Return the data to update Medusa's state.`  
  `return payment.data;`  
`}`

`async cancelPayment(`  
  `payment: Payment`  
`): Promise<PaymentProcessorError | Record<string, unknown>> {`  
  `// No on-chain action is possible. Just return the current state.`  
  `return payment.data;`  
`}`

This complete Medusa.js payment processor successfully adapts the platform's architecture to the unique, asynchronous flow of Bitrequest, creating a robust and stateful system ready for the final, critical step: server-side verification.

## **Section 4: The Imperative of Server-Side Verification**

This section addresses the most critical aspect of the entire integration: the secure, server-side verification of every transaction. The architectural design of Bitrequest, which relies on a client-side notification, mandates a "distrust and verify" security posture. Relying solely on the data received from the result\_callback function would expose the e-commerce store to trivial fraud, where a malicious actor could simulate the callback and receive goods without ever making a payment. The process detailed here transforms the untrusted client-side signal into a definitive, auditable, and secure confirmation of payment by querying the blockchain ledger directly.

### **4.1 The Golden Rule: Never Trust Client-Side Data**

A fundamental principle of web security is that any data originating from a client's browser must be considered untrusted until it is independently verified by the server. In the context of this integration, the post\_data object received by the result\_callback is client-side data. A technically proficient user could easily use their browser's developer tools to manually invoke this function with fabricated data, such as a fake transaction hash and a status: "paid" message.  
If the system were to fulfill an order based on this message alone, it would be fundamentally broken. Therefore, the server-side verification process is not an optional enhancement; it is a non-negotiable security requirement. The data from the callback (txhash, receiver, amount, pid) should be used only as pointers—information that helps the server find the transaction on the blockchain. The blockchain itself is the only source of truth.

### **4.2 A Multi-Blockchain Verification Playbook**

To handle the variety of cryptocurrencies supported by Bitrequest , the backend must be equipped with a verification service capable of communicating with different blockchain explorers and node APIs. This service will act as a router, taking the transaction data and payment details, and delegating the verification to the appropriate method based on the currency used.  
The following table provides a high-level overview of the recommended APIs and key checks for several major cryptocurrencies.

| Cryptocurrency | Recommended API Provider | API Endpoint Example | Key Verification Checks |
| :---- | :---- | :---- | :---- |
| Bitcoin | Blockstream.info (Esplora) | GET /api/tx/{txhash} | 1\. Find output (vout) to merchant address. 2\. Verify output value \>= expected amount. 3\. Check status.confirmed is true (or accept 0-conf based on risk tolerance). |
| Ethereum (ETH) | Etherscan / Alchemy | eth\_getTransactionByHash | 1\. Verify to address matches merchant address. 2\. Verify value (in Wei) \>= expected amount. 3\. Check transaction is included in a mined block. |
| ERC-20 Token | Etherscan / Alchemy | account module, tokentx action | 1\. Find Transfer event log. 2\. Verify contractAddress matches the token. 3\. Verify to address matches merchant address. 4\. Verify value \>= expected amount. |
| Lightning (LND) | Merchant's LND Node | GET /v1/invoice/{r\_hash} | 1\. Look up invoice by payment hash. 2\. Verify settled field is true. 3\. Verify amt\_paid\_sat \>= expected amount. |
| Lightning (LNbits) | Merchant's LNbits Instance | GET /api/v1/payments/{payment\_hash} | 1\. Check payment details by hash. 2\. Verify the paid field is true. |
| Nano | Public Nano Node (e.g., SomeNano) | account\_history or block\_info | 1\. Fetch block details using the hash. 2\. Verify block type is send. 3\. Verify account (destination) matches merchant address. 4\. Verify amount (in raw) \>= expected amount. |

#### **4.2.1 Verifying On-Chain Transactions (Bitcoin)**

For a proof-of-work chain like Bitcoin, verification involves fetching the transaction details from a trusted block explorer API and validating its outputs and confirmation status. The open-source Esplora API, hosted by services like Blockstream.info, is an excellent choice.  
**Steps for Bitcoin Verification:**

1. Receive the txhash from the callback.  
2. Make a GET request to https://blockstream.info/api/tx/{txhash}.  
3. Upon receiving a successful JSON response, perform the following checks:  
   * **Address Match:** Iterate through the transaction's outputs (vout array). Find the output where the scriptpubkey\_address matches the merchant's receiving address for that order.  
   * **Amount Match:** For the matching output, confirm that its value (in satoshis) is greater than or equal to the expected payment amount. A small tolerance for underpayment may be configured, but exact or overpayment is the secure default.  
   * **Confirmation Status:** Check the status.confirmed field. If true, the transaction is included in at least one block and can be considered secure. For high-value orders, a check for the number of confirmations (current\_height \- status.block\_height \+ 1\) may be required to mitigate risks from blockchain reorganizations. Accepting zero-confirmation (0-conf) transactions is faster but carries a higher risk of double-spending.

`// Part of a hypothetical VerificationService class`  
`import axios from 'axios';`

`async verifyBitcoinTransaction(txHash: string, expectedReceiver: string, expectedAmountSatoshis: number): Promise<boolean> {`  
  `try {`  
    ``const { data: tx } = await axios.get(`https://blockstream.info/api/tx/${txHash}`);``

    `const output = tx.vout.find(o => o.scriptpubkey_address === expectedReceiver);`

    `if (!output) {`  
      ``console.error(`Verification failed: No output found for address ${expectedReceiver}`);``  
      `return false;`  
    `}`

    `if (output.value < expectedAmountSatoshis) {`  
      ``console.error(`Verification failed: Amount mismatch. Expected ${expectedAmountSatoshis}, got ${output.value}`);``  
      `return false;`  
    `}`

    `// For production, you might want to check for more than 1 confirmation`  
    `if (!tx.status.confirmed) {`  
      ``console.warn(`Verification warning: Transaction ${txHash} is not yet confirmed.`);``  
      `// Depending on risk tolerance, you might accept this or wait.`  
      `// For this guide, we require confirmation.`  
      `return false;`  
    `}`

    `return true;`  
  `} catch (error) {`  
    ``console.error(`Error verifying Bitcoin transaction ${txHash}:`, error);``  
    `return false;`  
  `}`  
`}`

#### **4.2.2 Verifying ERC-20 and L2 Token Transfers**

Verifying payments on Ethereum and EVM-compatible chains requires a different approach, especially for ERC-20 tokens, as their transfers are recorded in event logs rather than the transaction's top-level to and value fields. Etherscan provides a comprehensive API for this purpose.  
**Steps for ERC-20 Verification:**

1. Receive the txhash from the callback.  
2. Use the Etherscan API endpoint for "Get a list of 'ERC20 \- Token Transfer Events' by Address" or, more directly, get the transaction receipt and parse its logs.  
3. Using the transaction receipt (eth\_getTransactionReceipt), iterate through the logs array.  
4. For each log, check the following:  
   * **Contract Address:** The address of the log must match the contract address of the expected ERC-20 token.  
   * **Event Signature:** The first topic (topics) must match the Keccak-256 hash of the Transfer(address,address,uint256) event signature.  
   * **Recipient:** The third topic (topics\[span\_0\](start\_span)\[span\_0\](end\_span)), representing the to address, must be decoded and matched against the merchant's receiving address.  
   * **Amount:** The data field of the log, representing the value, must be decoded and confirmed to be greater than or equal to the expected amount.

#### **4.2.3 Verifying Lightning Network Payments**

Lightning payments are off-chain, so public block explorers cannot be used for verification. Instead, the merchant must query their own Lightning node.

* **For LND users:** The LND REST API provides an endpoint to look up an invoice by its payment hash (r\_hash). A GET request to /v1/invoices/{r\_hash} (with the hash hex-encoded) will return a JSON object. The critical fields to check are settled: true and amt\_paid\_sat, which must match or exceed the expected amount.  
* **For LNbits users:** LNbits offers a simpler REST API. A GET request to /api/v1/payments/{payment\_hash} on the merchant's LNbits instance will return the payment details. The key is to check for "paid": true in the response.

#### **4.2.4 Verifying Nano Transactions**

Nano uses a block-lattice architecture where each account has its own blockchain. Transactions are near-instantaneous and feeless. Verification can be done by querying a public Nano node.  
**Steps for Nano Verification:**

1. Receive the txhash (which is a block hash in Nano) from the callback.  
2. Make a POST request to a public RPC endpoint (e.g., https://node.somenano.com/proxy) with the action block\_info and the hash.  
3. Parse the response and check:  
   * The contents.link\_as\_account field, which is the destination address, must match the merchant's receiving address.  
   * The amount (in raw, the smallest unit of Nano) must be greater than or equal to the expected amount.  
   * The confirmed field should be true.

### **4.3 Ensuring Idempotency with the Payment ID (pid)**

The Bitrequest callback might, under certain network conditions, be triggered more than once for the same payment. The backend system must be resilient to this. The pid (the Medusa payment\_session.id) is the key to achieving this idempotency.  
Before initiating the verification process for any incoming callback, the logic in the custom API endpoint (from Section 3.3) must first check the status of the cart or order associated with that pid.

1. Retrieve the cart using the cart\_id and payment\_session\_id.  
2. Check if the cart has already been completed (cart.completed\_at is not null).  
3. If it has, the payment has already been verified and processed. The endpoint should immediately return a success response without re-running the verification logic. This prevents race conditions and ensures an order is never fulfilled twice for a single payment.

## **Section 5: Advanced Configurations and Best Practices**

Moving from a functional prototype to a production-grade payment system involves hardening security, enhancing user privacy, and implementing robust operational practices. This final section covers advanced configurations and a checklist of best practices to ensure the Bitrequest integration is reliable, secure, and easy to manage.

### **5.1 Strategies for Managing Multiple Cryptocurrency Addresses**

Reusing a single cryptocurrency address for all incoming payments is a significant privacy leak. It allows third parties to easily track a merchant's total revenue and transaction history on the public blockchain. Bitrequest directly supports a solution to this problem through its implementation of BIP44 and extended public keys (xPubs).  
An xPub is a master public key from which a virtually limitless number of new, unique public addresses can be derived without exposing the corresponding private keys. By integrating xPubs, the Medusa.js backend can generate a fresh address for every single payment session.  
**Implementation Strategy:**

1. **Store the xPub:** Securely store the xPub key for each cryptocurrency (e.g., Bitcoin, Litecoin) in the Medusa.js backend's configuration or a secure vault.  
2. **Use a Derivation Library:** Incorporate a library like bip32 or bitcoinjs-lib into the Medusa.js backend.  
3. **Derive in initiatePayment:** Within the BitrequestPaymentService's initiatePayment method, instead of using a static address, use the derivation library to generate a new address from the xPub. A simple approach is to use an incrementing index stored with the payment provider's settings. For example, derive the address at path m/0/0 for the first payment, m/0/1 for the second, and so on.  
4. **Pass to Frontend:** The newly derived address is then included in the session\_data sent to the frontend, ensuring each customer pays to a unique address.

This practice dramatically improves the financial privacy of both the merchant and their customers.

### **5.2 Leveraging Fiat-Pegged Pricing to Mitigate Volatility**

One of the most significant challenges for merchants accepting cryptocurrency is price volatility. The value of a crypto asset can change dramatically in the time between a customer checking out and their transaction being confirmed.  
Bitrequest elegantly solves this with its mandatory uoa (Unit of Account) parameter. When a payment request is generated for, say, $99.95 USD, Bitrequest locks in that fiat value. It uses its real-time exchange rate data, which is updated every 10 minutes from sources like CoinMarketCap and CoinGecko, to calculate the precise amount of cryptocurrency the customer must send at the moment of payment.  
If the price of Bitcoin drops while the customer is completing the payment, the amount of BTC they are asked to send will automatically increase to match the $99.95 value. This effectively transfers the short-term volatility risk from the merchant to the customer (or the payment window), ensuring the merchant receives the correct fiat-equivalent value for their goods. This feature is enabled by default through the correct construction of the payment URL and requires no additional configuration beyond specifying the uoa and amount in a fiat currency.

### **5.3 A Production-Ready Deployment Checklist**

Before deploying the integration to a live environment, a thorough review against a production checklist is essential to ensure security, reliability, and maintainability.

* **Security:**  
  * **API Key Management:** All API keys for third-party verification services (Etherscan, Blockstream, etc.) and Lightning nodes must be stored as environment variables or in a dedicated secret management service (e.g., AWS Secrets Manager, HashiCorp Vault). They must never be hardcoded in the source code.  
  * **Endpoint Protection:** The custom Medusa.js callback endpoint (/store/bitrequest/callback) should be protected by any authentication or session management middleware that the rest of the storefront API uses to prevent unauthorized access.  
  * **Input Validation:** Rigorously validate all data received at the callback endpoint. Ensure tx\_data and other fields conform to expected formats before processing.  
* **Error Handling and Reliability:**  
  * **Verification Service Downtime:** The VerificationService must have robust error handling. What happens if the Blockstream.info API is temporarily unavailable? The system should not fail the payment immediately. Implement a retry mechanism with exponential backoff. If verification repeatedly fails due to an external service outage, the payment should be flagged for manual review, and an alert should be sent to the operations team.  
  * **Frontend Error States:** The frontend checkout flow must gracefully handle error responses from the backend verification API. If the backend returns a 400 or 500 error, the UI should display a clear message to the user (e.g., "Payment verification failed. Please contact support.") instead of getting stuck in a "verifying" state.  
  * **Logging:** Implement comprehensive logging at every stage of the payment and verification process. Log the received callback data, the start and end of verification attempts, the results from explorer APIs, and the final success or failure status. This is invaluable for debugging issues.  
* **Testing and Monitoring:**  
  * **Idempotency Testing:** Actively test the idempotency logic by sending multiple identical callback requests for the same pid to the backend endpoint. Verify that the order is created only once.  
  * **Pending Payment Monitoring:** Create a scheduled job or monitoring query that identifies payment sessions that have been initiated but have not been authorized or canceled within a reasonable timeframe (e.g., 24 hours). This can help detect cases where a customer paid, but the Bitrequest callback failed to fire or was not received by the application. These cases can then be investigated and resolved manually.

By adhering to these advanced configurations and best practices, developers can elevate the Bitrequest integration from a basic functional implementation to a secure, resilient, and professional e-commerce payment solution.

#### **Works cited**

1\. bitrequest/bitrequest.github.io: Web Application for crypto ... \- GitHub, https://github.com/bitrequest/bitrequest.github.io 2\. bitrequest/webshop-integration \- GitHub, https://github.com/bitrequest/webshop-integration 3\. Payment Architecture Overview \- Medusa Docs, https://docs.medusajs.com/v1/modules/carts-and-checkout/payment 4\. How to Create a Payment Processor \- Medusa Documentation, https://docs.medusajs.com/v1/modules/carts-and-checkout/backend/add-payment-provider 5\. Payment \- Medusa Documentation, https://docs.medusajs.com/resources/commerce-modules/payment/payment 6\. Payment Module Data Models Reference \- Medusa Documentation, https://docs.medusajs.com/resources/references/payment/models/Payment 7\. How do I look up my Bitcoin transaction? \- Blockstream, https://blog.blockstream.com/education/wallets/how-do-i-look-up-my-bitcoin-transaction/ 8\. esplora-client \- NPM, https://www.npmjs.com/package/esplora-client 9\. Explorer API · Bitcoin Explorer \- Blockstream.info, https://blockstream.info/explorer-api 10\. Get An Address's Full Transaction History \- Etherscan API, https://docs.etherscan.io/etherscan-v2/get-an-addresss-full-transaction-history 11\. Accounts \- Etherscan API, https://docs.etherscan.io/etherscan-v2/api-endpoints/accounts 12\. Tokens \- Etherscan API, https://docs.etherscan.io/etherscan-v2/api-endpoints/tokens 13\. ListInvoices \- LND | Lightning Labs API Reference, https://api.lightning.community/api/lnd/lightning/list-invoices/index.html 14\. LookupInvoice | Lightning Labs API Reference, https://lightning.engineering/api-docs/api/lnd/lightning/lookup-invoice/ 15\. LNbits Documentation \- GitHub, https://github.com/lnbits/lnbits/wiki/LNbits-Documentation?ref=nobsbitcoin.com 16\. Developing with LNbits \- GitHub, https://github.com/lnbits/lnbits/wiki/Developing-with-LNbits 17\. SomeNano Node | A free-to-use Public Nano Node since 2021, https://node.somenano.com/ 18\. Connect to Nano (XNO) Node and Block Explorer \- NOWNodes, https://nownodes.io/nodes/nano-xno 19\. Retrieving Nano Wallet Information via Nano RPC Full Node \- A Complete Tutorial, https://nownodes.io/blog/get-nano-wallet-information-tutorial/