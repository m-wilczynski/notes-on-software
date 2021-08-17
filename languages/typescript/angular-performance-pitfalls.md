[Typescript](/languages/typescript)
# Angular performance pitfalls

Common Angular performance pitfalls to be vary of, when meeting FPS/perf issues.

**Contents**
- [Binding functions and properties to view](/languages/typescript/angular-performance-pitfalls?id=binding-functions-and-properties-to-view)
- [Loading all the modules at once](/languages/typescript/angular-performance-pitfalls?id=loading-all-the-modules-at-once)
- [Not cleaning up subscriptions](/languages/typescript/angular-performance-pitfalls?id=not-cleaning-up-subscriptions)
- [Not caching slowly changing (or static) responses from HTTP calls](/languages/typescript/angular-performance-pitfalls?id=not-caching-slowly-changing-or-static-responses-from-HTTP-calls)
- [Not using `OnPush` change detection strategy for hot paths](/languages/typescript/angular-performance-pitfalls?id=not-using-OnPush-change-detection-strategy-for-hot-paths)
- [Not offloading long running tasks to WebWorkers](/languages/typescript/angular-performance-pitfalls?id=not-offloading-long-running-tasks-to-webWorkers)

#### Binding functions and properties to view

Since Angular has no chance to detect when output of function or property changes, it needs to schedule (to zone.js) and continously check, whether it has changed or not (and should it update the view).

**Example (bad):**
```typescript
//my.component.ts
public class MyComponent {

    private _myText: string;

    public get myText(): string { return this._myText.ToUpperCase(); }
}
```
```html
<!-- my.component.html -->
<div>
    {{myText}}
</div>
```

**Example (good):**
```html
<!-- Given ToUpperCasePipe is implemented -->
<div>
    {{myText | toUpperCase}}
</div>
```

If the case is only to format (or transform in any other way) value of a variable, some sort of `pipe` could be used (`DatePipe` is a builtin example of such case). If it was meant to be an encapsulation attempt, to only allow view to read (but not mutate) variable, decision should be made - what is more important in this place? Is it hotpath? Do I really need to encapsualate the value and pay for it with (possibly) worse performance?

<br> 

#### Loading all the modules at once

Another pitfall would be to attempt to fetch all of the modules on application start at once. Angular supports lazy loading modules on navigation, so if there is a chance user would not use part of an app or doesn't even have permissions to access it, there would be much less data required to fetch from the server and less CPU cycles to use. In most cases this should lead to far better user experience - of course if we do not push everyting to lazy modules. If there is starting module, that only has navigation, that could possibly lead to user frustration if not handled properly.

**Example:**
```typescript
const routes: Routes = [
    {
        //Lazy load administration.module on this route
        path: 'administration',
        loadChildren: () => import('./administration/administration.module.ts')
            .then(m => m.AdministrationModule),
        //bonus points on activating route only if user has permissions
        canActivate: [AdministrationPermissionGuard]
    }
];
```
<br>

#### Not cleaning up subscriptions

Every subscription `myObservable$.subscribe(_ => ...)` should be cleaned up on component destroy. To achieve that, we can either do it manually, with RxJs `takeUntil` or with `async` pipe.
Although some APIs cleanup their subscriptions (like `HttpClient`), general rule of a thumb would be to set up cleanup in code anyway.

**Example (manual):**
```typescript
public class MyComponent implements OnDestroy {

    private _subscriptions[]: Subscription[] = [];

    //Called onInit, on click etc.
    subscribeToChanges() {
        this._subscriptions.push(
            this.someObservable$.subscribe(_ => ...),
            this.anotherObservable$.subscribe(_ => ...)
        ];
    }

    onDestroy() {
        _subscription.forEach(_ => _.unsubscribe());
    }

}
```

**Example (takeUntil):**
```typescript
public class MyComponent implements OnDestroy {

    private _onDestroy$: Subject<any> = new Subject<any>();

    //Called onInit, on click etc.
    subscribeToChanges() {
        this.someObservable$.pipe(
            takeUntil(this._onDestroy$)
        ).subscribe(_ => ...);

        this.anotherObservable$.pipe(
            takeUntil(this._onDestroy$)
        ).subscribe(_ => ...);
    }

    onDestroy() {
        _onDestroy$.next({});
        _onDestroy$.complete();
    }

}
```

**Example (async pipe):**
```html
<div>
    {{someObservable$ | async}}
</div>
```
<br>

#### Not caching slowly changing or static responses from HTTP calls

That's a common case for both frontend and backend side of the fence. If the queried data is slowly changing or is static (ie. dictionaries, that are extremely unlikely to change during app use), we should cache it - the earlier in the pipeline, the better.

**Example (static data):**
```typescript
public class MyService {

    private _myResponse: MyResponse;

    public getData(): Observable<MyResponse> {
        if (this._myResponse) { return of(this._myResponse); }
        return this.http.get<MyResponse>('api/my-endpoint/')
            .pipe(
                //Cache the response here
                tap(_ => this._myResponse = _)
            );
    }
}
```

**Example (simple time cache):**
```typescript
//time-cache.ts
public class TimeCache<T> {
    private _cachedObject: T;
    private _cachedUntil: Date;

    public get value(): T {
        if (new Date() <= this._cachedUntil) {
            return this._cachedObject;
        }
        return null;
    }

    constructor(objectToCache: T, cacheTimeInSec: number) {
        //Some validations should take place here ofc
        this._cachedObject = objectToCache;
        this._cachedUntil = new Date(
            new Date().getTime() + 1000 * cacheTimeInSec
        );
    }
}

//my.service.ts
public class MyService {
    private _myResponse: TimeCache<MyResponse>;
    public const cacheTimeInSec = 15;

    public getData(): Observable<MyResponse> {
        if (this._myResponse && this._myResponse.value) { 
            return of(this._myResponse.value); 
        }

        return this.http.get<MyResponse>('api/my-endpoint/')
            .pipe(
                //Cache the response here
                tap(_ => this._myResponse = 
                    new TimeCache<MyResponse>(_, this.cacheTimeInSec);
                )
            );
    }
}
```
<br>

#### Not using `OnPush` change detection strategy for hot paths

Default Angular changed detection strategy (`ChangeDetectionStrategy.Default`) is based on mutability of object's state. For most programmers comming from object-oriented programming such approach is very handy to work with. It comes with a cost though. If we have a view, that represents a list of objects, each having multiple props, we have OxP possibilities for this view to change (O - object count, P - props count on each object); this is of course quite large simplification but does represent cost of default change detection strategy.

To solve this issue, `ChangeDetectionStrategy.OnPush` strategy has been introduced.
It works by comparing only objects references of objects that are passed to an object via `@Input`. Since only references are checked, it is highly recommended to use **immutable** objects with this strategy, or else it will get messy pretty soon. Quick recall: all primitives in JavaScript are immutable too.
OnPush strategy works also with `Observable<T>` but only when bound with `| async` pipe in view.

If we want to be exact when change detection should fire, we could force it with `ChangeDetectionRef.markForCheck()`.

**Example (both cases of OnPush strategy):**
```typescript
@Component(
    selector: 'my-component',
    changeDetection: ChangeDetectionStrategy.OnPush,
    template: '
        <div>
            {{myObservable$.text | async}}
        </div>
        <div>
            {{myInput.text}}
        </div>
    '
)
public class MyComponent {

    myObservable$: Observable<MyObject>;

    //Received from parent.
    //To trigger, should receive new instance each time, 
    //ie. { text: 'My new text!' }
    @Input() myInput: MyObject;
}

public class MyObject {
    public text: string;
}
```
<br>

#### Not offloading long running tasks to WebWorkers

Another case that has similarities on both backend and frontend land. If there is a process that is meant to be long running, we should consider offloading it to background thread. On client side we can use **WebWorkers** to achieve this. They can be scaffolded with angular/cli as any other framework construct with `ng g web-worker background`.

**Example:**
```typescript
//background.worker.ts
addEventListener('message' ({ data }) => {
    //do something with 'data' and post result
    postMessage(/*return response from worker*/);
});

//some.component.ts
public class SomeComponent {
    offloadComputation() {
        //Check feature support; can also use "if (window.Worker)"
        if (typeof Worker !== 'undefined' ) {
            const myWorker = 
                new Worker('/background.worker.ts', { type: 'module' });
                //Set callback, when worker uses "postMessage"
                worker.onmessage = ({ data }) =>  {
                    //Received message from worker
                };

                //Send data to worker, ie. offload some work
                worker.postMessage({ workToDo: data });
        } else {
            //Unsupported, fallback
        }
    }
}
```