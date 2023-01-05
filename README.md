# Create a DynamoDB Table in SST
We are now going to start creating our infrastructure in SST using AWS CDK. Starting with DynamoDB.

## Create a Stack
 Add the following to a new file in ```stacks/StorageStack.js```
 ```sh
import { Table } from "@serverless-stack/resources";

export function StorageStack({ stack, app }) {
  // Create the DynamoDB table
  const table = new Table(stack, "Notes", {
    fields: {
      userId: "string",
      noteId: "string",
    },
    primaryIndex: { partitionKey: "userId", sortKey: "noteId" },
  });

  return {
    table,
  };
}
```
Let’s quickly go over what we are doing here.

We are creating a new stack in our SST app. We’ll be using it to create all our storage related infrastructure (DynamoDB and S3). There’s no specific reason why we are creating a separate stack for these resources. It’s only meant as a way of organizing our resources and illustrating how to create separate stacks in our app.

We are using SST’s ```Table``` construct to create our DynamoDB table.

It has two fields:

1. ```userId:``` The id of the user that the note belongs to.
2. ```noteId:``` The id of the note.
We are then creating an index for our table.

Each DynamoDB table has a primary key. This cannot be changed once set. The primary key uniquely identifies each item in the table, so that no two items can have the same key. DynamoDB supports two different kinds of primary keys:
<ul>
<li> Partition key</li>
<li>Partition key and sort key (composite)</li>
</ul>
We are going to use the composite primary key (referenced by primaryIndex in code block above) which gives us additional flexibility when querying the data. For example, if you provide only the value for userId, DynamoDB would retrieve all of the notes by that user. Or you could provide a value for userId and a value for noteId, to retrieve a particular note.

We are also returning the Table that’s being created publicly.
```sh
return {
  table,
};
```
This’ll allow us to reference this resource in our other stacks.

Note, learn more about sharing resources between stacks here.

## Remove Template Files
The Hello World API that we previously created, can now be removed. We can also remove the files that came with the starter template.

 To remove the starter stack, run the following from your project root.
```sh
$ npx sst remove MyStack
```
This will take a minute to run.

 Also remove the template files.
```sh
$ rm stacks/MyStack.js services/functions/lambda.js
```
## Add to the App
Now let’s add our new stack to the app.

 Replace the ```stacks/index.js``` with this.
```sh
import { StorageStack } from "./StorageStack";

export default function main(app) {
  app.setDefaultFunctionProps({
    runtime: "nodejs16.x",
    srcPath: "services",
    bundle: {
      format: "esm",
    },
  });
  app.stack(StorageStack);
}
```
## Deploy the App
If you switch over to your terminal, you’ll notice that you are being prompted to redeploy your changes. Go ahead and hit ENTER.

Note that, you’ll need to have ```sst start``` running for this to happen. If you had previously stopped it, then running ```npx sst start``` will deploy your changes again.

You should see something like this at the end of the deploy process.
```sh
Stack dev-notes-StorageStack
  Status: deployed
```
The ```Stack``` name above of `dev-notes-StorageStack` is a string derived from your `${stageName}-${appName}-${stackName}`. Your `appName` is defined in the `name` field of your `sst.json` file and your `stackName` is the function name you choose for your stack in `stacks/StorageStack.js`.

You can also head over to the <B> DynamoDB </B> tab in the SST Console and check out the new table.

<img src="https://imgur.com/8Ps8S5h.png" alt="Homepage view 3" width=1000 height=500/>

Now that our database has been created, let’s create an S3 bucket to handle file uploads.

# Create an S3 Bucket in SST

We’ll be adding to the StorageStack that we created.

## Add to the Stack
 Add the following above the `Table` definition in `stacks/StorageStack.js`.
```sh
// Create an S3 bucket
const bucket = new Bucket(stack, "Uploads");
```
 Make sure to import the `Bucket` construct. Replace the import line up top with this.
```sh
import { Bucket, Table } from "@serverless-stack/resources";
```
This creates a new S3 bucket using the SST `Bucket` construct.

Also, find the following line in `stacks/StorageStack.js`.
```sh
return {
  table,
};
```
 And add the `bucket` below `table`.
```sh
bucket,
```
This’ll allow us to reference the S3 bucket in other stacks.

Note, learn more about sharing resources between stacks here.

# Deploy the App
If you switch over to your terminal, you’ll notice that you are being prompted to redeploy your changes. Go ahead and hit ENTER.

Note that, you’ll need to have `sst start` running for this to happen. If you had previously stopped it, then running `npx sst start` will deploy your changes again.

You should see that the storage stack has been updated.
```sh
Stack dev-notes-StorageStack
  Status: deployed
  ```
You can also head over to the <b>Buckets</B> tab in the SST Console and check out the new bucket.

<img src="https://imgur.com/KNtCCXR.png" alt="You can also head over to the <b>Buckets</B> tab in the SST Console and check out the new bucket." width=1000 height=500/>

## Commit the Changes
 Let’s commit and push our changes to GitHub.
```sh
$ git add .
$ git commit -m "Adding a storage stack"
$ git push
```
Next, let’s create the API for our notes app.
