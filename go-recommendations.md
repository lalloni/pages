# Go recommendatios

## Logging library

For any serious production ready system the std lib logging library lacks key features.
Using a library which implements leveled and structured logging is a must.
For performance reasons I recommend using [Uber'z zap](https://github.com/uber-go/zap) above other (good) alternatives that exist. *Please learn how to use it right.*
