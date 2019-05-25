[Angular](/frameworks-and-libraries/angular)
# Combining promises with observables

Recently I've stumble upon a problem in combining `Promise` and `Observable` in Angular code.
It's not an Angular problem per se (or not even TypeScript problem) but general RxJs and ES6 promises combination.

Case description:
- we retrieve result from async call to REST API via `HttpClient` (result is `Observable`)
- result from API are settings needed for building an object coming from external library; 
- after object is built, we call its function that represents an async operation but returns `Promise`
- finally we need to do something with result of the `Promise` and project result to observer

What we need to do is combine `pipe` with converting `Promise` to `Observable` using `from` and flattening `Observable<Observable<T>>` with `flatMap`;

Code:
```typescript
getResultFromExternalLib() : Observable<Result> { 

    return this.httpClient.get(this.settingsUrl)
        .pipe(flatMap(settings => {
            const externalLibObject = new ExternalLibObject(settings);
            return from(externalLibObject.asyncOperation()
                .then(promiseResult => projectFinalResultFrom(promiseResult)));
    }));
}

```