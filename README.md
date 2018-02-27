# bs-aws-lambda

`bs-aws-lambda` is a set of types to use when creating [AWS
lambda](http://docs.aws.amazon.com/lambda) handlers.

Those types are inspired by the
[`@types/aws-lambda`](https://www.npmjs.com/package/@types/aws-lambda)
package with typescript types.

## Installation

```
yarn add @ahrefs/bs-aws-lambda
```

or to follow the master version:

```
yarn add https://github.com/ahrefs/bs-aws-lambda.git
```

## Usage

Add `@ahrefs/bs-aws-lambda` to the `bs-dependencies` of `bsconfig.json`.

Then when you create a function handler for a lambda, you can annotate
it with one of `AwsLambda.Scheduled.handler`,
`AwsLambda.Dynamodb.streamHandler`, `AwsLambda.Sns.handler`, etc. This
way you are sure to expose a function which respect the signature
expected by Lambda and you get types information for all the
parameters this function will receive.

## Example

Simple echo function for the API Gateway. This might seem a little
verbose compare to a typescript version. But using this package, you
are sure to handle all the possible cases where a value is actually
null or base64 encoded. Plus the returned object given to the callback
will always have the expected fields and types.

```reason
[@bs.send.pipe : Node.Buffer.t] external bufferToString : string = "toString";

let echo: AwsLambda.APIGatewayProxy.handler =
  (event, _context, cb) => {
    let parameter =
      Js.Null.toOption(event##queryStringParameters)
      |> Js.Option.andThen([@bs] (params => Js.Dict.get(params, "userid")));
    switch parameter {
    | Some(userid) => Js.log2("executing lambda for", userid)
    | None => Js.log("executing lambda for anonymous user")
    };
    let result =
      switch (Js.Null.toOption(event##body)) {
      | None =>
        Js.log("error: no body available in the request");
        AwsLambda.APIGatewayProxy.result(
          ~body=`Plain({|{"status": "no body available in the request"}|}),
          ~statusCode=400,
          ()
        );
      | Some(body) => {
          "body": body,
          "statusCode": 200,
          "headers": Js.Nullable.null,
          "isBase64Encoded": Js.Nullable.return(event##isBase64Encoded)
        }
      };
    cb(Js.null, result);
    Js.Promise.resolve();
  };
```

The typescript equivalent of the type annotation is actually more verbose:

```typescript
export const handle: Lambda.APIGatewayProxyHandler = async (
    event: Lambda.APIGatewayEvent,
    context: Lambda.Context,
    cb: Lambda.APIGatewayProxyCallback) => {
}
```
