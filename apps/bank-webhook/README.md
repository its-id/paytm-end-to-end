## Setting up the WebHook Handler

1. Create a new folder called `bank_webhook_handler` in the `apps` directory.

2. Initialize a new Node.js project in the `bank_webhook_handler` directory.

   ```bash
   npm init -y
   ```

3. Install the following dependencies:

   ```bash
   npm i esbuild express @types/express
   ```

4. Create and setup the `tsconfig` file.

   ```json
   {
     "extends": "@repo/typescript-config/base.json",
     "compilerOptions": {
       "outDir": "dist"
     },
     "include": ["src"],
     "exclude": ["node_modules", "dist"]
   }
   ```

5. Create the `src/index.ts` file and add the following code:

   ```typescript
   import express from 'express';

   const app = express();

   app.post('/hdfcWebhook', (req, res) => {
     //TODO: Add zod validation here?

     const paymentInformation = {
       token: req.body.token,
       userId: req.body.user_identifier,
       amount: req.body.amount,
     };

     // Update balance in db, add txn
   });
   ```

6. Update the Prisma Schema in `packages/db/prisma/schema.prisma` to include two things in relation to the user:

   - Balance
   - Transaction

   ```prisma
   ...

   model User {
        id                Int                 @id @default(autoincrement())
        email             String?             @unique
        name              String?
        number            String              @unique
        password          String
        OnRampTransaction OnRampTransaction[]
        Balance           Balance[]
   }

   model OnRampTransaction {
       id        Int          @id @default(autoincrement())
       status    OnRampStatus
       token     String       @unique
       provider  String
       amount    Int
       startTime DateTime
       userId    Int
       user      User         @relation(fields: [userId], references: [id])
   }

   model Balance {
       id     Int  @id @default(autoincrement())
       userId Int  @unique
       amount Int
       locked Int
       user   User @relation(fields: [userId], references: [id])
   }

   enum OnRampStatus {
       Success
       Failure
       Processing
   }
   ...
   ```

7. Open `packages/db` in terminal and run the following command to migrate and generate the Prisma Client:

   ```bash
   npx prisma migrate dev --name added_balanced_and_onramp
   npx prisma generate
   ```

8. Add `db` as a dependency in the `bank_webhook_handler/package.json`.
   ```json
   ...
   "dependencies": {
   "@repo/db": "^1.0.0",
   ...
   }
   ...
   ```

## Completing the WebHook Logic

1. When a request is raised, we need to update the balance of the user and add a transaction to the database.

2. We perform two operations inside a transaction to ensure that both the balance and transaction are updated together.

   ```typescript
   ...
     await db.$transaction([
       db.balance.upsert({
         where: { userId: paymentInformation.userId },
         update: {
           amount: {
             increment: paymentInformation.amount,
           },
         },
       }),
       db.onRampTransaction.updateMany({
        where: {
          token: paymentInformation.token,
        },
        data: {
          token: paymentInformation.token,
          provider: 'HDFC',
          amount: Number(paymentInformation.amount),
          status: 'Success',
          userId: Number(paymentInformation.userId),
        },
      }),
     ]);
   ...
   ```

3. If any of the two operation fails, the transaction will be rolled back.
