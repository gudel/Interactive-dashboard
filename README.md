## Next.js App Router Course - Starter

This is the starter template for the Next.js App Router Course. It contains the starting code for the dashboard application.

For more information, see the [course curriculum](https://nextjs.org/learn) on the Next.js Website.

# running the SPA locally

1. initialize pnpm
```pnpm i```
2. run development server locally
```pnpm dev```

# Repository structure

This repository functions as both a standalone main repo and a Git submodule for broader project integration and tracking.

# Database used within the project

Neon serverless Postgres

# Hurdles

Differentiated from debug since this is more of an abstracted problem.

1. Difficulty grasping SQL logic. Chapter 7 of the tutorial jumps around too fast.
It would seem that it assumes everything will run fine on the learner's end. Which is far from the truth.

# Debug log

1. Following the tutorial blindly resulted in a failed deployment on Vercel. After some deep issue tracking, particularly in [this discussion](https://github.com/vercel/next.js/discussions/76822) and [this stack overflow thread](https://stackoverflow.com/questions/76710159/error-while-deploying-nextjs-app-to-vercel), it turns out the issue stemmed from a dependency conflict.

Fix:
- Switched from bcrypt to bcryptjs
- Adjusted route.ts in /app/seed/

This resolved the issue and allowed for a successful deployment.

2. Database seeding sometimes requires repeat action. If the first seeding produces an error, try it again, two more times, to make sure it's not random error. This will save time. 
Remember, once is a coincidence, twice is a statistical 50/50, thrice is a surefire error.

3. Database tutorial at chapter 7 is ambiguous when compared to earlier writing style.
Make sure to follow and pay attention to the files being mentioned.
if you fall into a pitfall like me, retrace your steps, the issue will resolve since you need to comment out specific lines. To nake things clearer:

problem:
- Ambiguous instruction
- Assumed that it follows previous flow and style shown in earlier chapter
- Copied the correct code to the wrong file multiple times because the tutorial didn't specify clearly enough.

fix: 
- Retrace steps, make sure the files are synced with the one mentioned in the tutorial.
- ctrl+z is your friend. Don't be shy to use it.

4. On card data. Where I need to fetch a string of array. I fell into another pitfall of redundant const declaration.

problem: 
Need to call specific values from a return.

What I did:
```export default async function Page() {
  const revenue = await fetchRevenue();
  const latestInvoices = await fetchLatestInvoices(); 
  const totalPaidInvoices = await fetchCardData();          
  const totalPendingInvoices = await fetchCardData();       /* pitfall: redundant, wrong way to call multiple stuff from fetchCardData
  const numberOfInvoices = await fetchCardData();          
  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
        Dashboard
      </h1>
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        { <Card title="Collected" value={totalPaidInvoices} type="collected" /> }   /* need to call this
        { <Card title="Pending" value={totalPendingInvoices} type="pending" /> }    /* need to call this
        { <Card title="Total Invoices" value={numberOfInvoices} type="invoices" /> }    /* need to call this
        { <Card
          title="Total Customers"
          value={numberOfCustomers}
          type="customers"
        /> }
```

What I did was basically pushing myself to a giving up state and ends up just opening the solution to gain insight.

Fix: Call it as an array.

Since the declaration in /lib/data.tsx
is formed like this:

```
export async function fetchCardData() {
  try {
    // You can probably combine these into a single SQL query
    // However, we are intentionally splitting them to demonstrate
    // how to initialize multiple queries in parallel with JS.
    const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
    const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
    const invoiceStatusPromise = sql`SELECT
         SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS "paid",
         SUM(CASE WHEN status = 'pending' THEN amount ELSE 0 END) AS "pending"
         FROM invoices`;

    const data = await Promise.all([
      invoiceCountPromise,
      customerCountPromise,
      invoiceStatusPromise,
    ]);

    const numberOfInvoices = Number(data[0][0].count ?? '0');
    const numberOfCustomers = Number(data[1][0].count ?? '0');
    const totalPaidInvoices = formatCurrency(data[2][0].paid ?? '0');
    const totalPendingInvoices = formatCurrency(data[2][0].pending ?? '0');

    return {
      numberOfCustomers,
      numberOfInvoices,                             /* <-This is an array as a return
      totalPaidInvoices,
      totalPendingInvoices,
    };
```
Calling the functions as an array like so:

```
import {
  fetchRevenue,
  fetchLatestInvoices,
  fetchCardData,
} from '@/app/lib/data';
 
export default async function Page() {
  const revenue = await fetchRevenue();
  const latestInvoices = await fetchLatestInvoices();
  const {
    numberOfInvoices,
    numberOfCustomers,
    totalPaidInvoices,
    totalPendingInvoices,
  } = await fetchCardData();
```
Fixed the issue.

Key takeaway. See the SQL return in the code and call the whole array instead of destructuring it. Each array is to be treated as 1 destucture.