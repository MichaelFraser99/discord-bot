# Discord Bot
This is a basic template for getting started with a Discord bot. The library is designed to be deployed within AWS lambda behind an API Gateway. Remember these services cost money and scale to a very high degree so take care when deploying.

## Requirements
- go 1.20+
- terraform 1.4.x+
- an AWS account

## Getting Started
1. Clone the repo - I would recommend making your repository private to avoid accidentally committing any secrets for the world to see
2. Run `git remote add upstream git@github.com:MichaelFraser99/serverless-discord-bot.git` to add the upstream remote and ensure you can pull in updates
3. Run `go mod tidy` to download the dependencies
4. Configure your bot (see below)
5. Run `make build` to build the binary
6. Deploy the binary to AWS lambda (see deployment section below)
7. Register your bot with discord (see below)

## Interactions Endpoint
This bot leverages the Interaction Endpoint feature of discord bots

https://discord.com/developers/docs/getting-started#adding-interaction-endpoint-url

The endpoint for this bot is your API Gateway endpoint + `/interactions`. For example, if your API Gateway endpoint is `https://example.com` then your interactions endpoint is `https://example.com/interactions`

## Configuring the bot
This project contains a basic configuration struct which is passed into the handler during the lambda init phase. This struct contains the following fields:
```go
type BotConfig struct {
	PublicKey                  string
	ApplicationCommandHandlers map[string]func(ctx context.Context, applicationCommand ApplicationCommand) (InteractionResponse, error)
}
```

## Adding custom commands
Configuring your discord bot with your own commands is very simple. Inside main.go you will see an example of adding a command to a bot:
```go
config = internal.BotConfig{
    PublicKey: publicKey,
    ApplicationCommandHandlers: map[string]func(ctx context.Context, applicationCommand internal.ApplicationCommand) (internal.InteractionResponse, error){
        "poke": func(ctx context.Context, applicationCommand internal.ApplicationCommand) (internal.InteractionResponse, error) {
            return internal.InteractionResponse{
                Type: 4,
                Data: internal.InteractionResponseData{
                    Content: "Hello, world!",
                    TTS:     false,
                },
            }, nil
        },
    },
}
```
The above code snippet registers a command `poke` and provides a function which will be executed on command run.

Therefor, when a user runs the command `/poke` the bot will respond with "Hello, world!".

## Deployment
This project is designed to be deployed to AWS lambda. The top-level Makefile contains a target for building the binary and zipping it up ready for deployment. The following command will build the binary and zip it up ready for deployment:
```bash
make build
```
After building, you can deploy the binary to AWS lambda using the AWS CLI. First cd into the infrastructure directory and review the vars.tfvars file

This file contains the following variables:
```terraform
name = "bot-name"
public_key = "add-bot-public-key-here"
```
Configure the name and public key for your bot. The name is used to name the lambda function and the public key is used to verify the requests from discord. The public key for your bot can be found on your bots application page in the discord developer portal.

When you're ready to deploy, run the following commands
```bash
make init #initialises terraform
make plan #runs a terraform plan
make apply #applies the terraform plan
```
