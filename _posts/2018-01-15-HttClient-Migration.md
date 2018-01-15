---
layout: post
title:  "Migrate to Angular HttpClient"
date:   2018-01-15 17:56:33 +0300
categories: angular
---

Recently I had to migrate our Angular application from `Http` to `HttpClient`.

The `HttpClient` API was introduce in the version `4.3`. It is an improvement of the existing `Http` API and can now be found in the `@angular/common/http` package.
Here are some of the changes that affected my refactoring:

## Response object is JSON by default

Rather than parsing the response like this

```javascript
    this.http.get('https://example.com/users')
        .map(response => response.json())
        .subscribe(result => console.log(result));
```

We can do

```javascript
    this.http.get('https://example.com/users')
        .subscribe(result => console.log(result));
```

## Typed responses

Now, the HTTP methods are parameterized and the response will be of the same type

```javascript
   interface User {
    id: number;
    name: string;
   }
   
   this.http.get<User>('https://example.com/users/1')
    .subscribe(result => {
        console.log(result.id);
        console.log(result.name);
    })
```

## Interceptors

This is a new addition to the API which allows custom logic to be applied globally to requests.
One of the scenarios where we needed such behavior is to check if the session has expired and redirect to the login page.

We did this by extending the `Http` class and checking the error status for each request as follows:

```javascript
    @Injectable()
    export class AuthenticatedHttpService extends Http {
    
        constructor(backend: XHRBackend, defaultOptions: RequestOptions) {
            super(backend, defaultOptions);
        }
    
        request(url: string | Request, options?: RequestOptionsArgs): Observable<Response> {
            return super.request(url, options).catch((error: Response) => {
                // perform custom action based on error status
                return Observable.throw(error);
            });
        }
    }
``` 

With the new API, we can now create an interceptor which will be added to the request processing pipeline

```javascript
export class AuthenticatedHttpInterceptor implements HttpInterceptor {

    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<HttpEventType.Response>> {
        return next.handle(req).do((event: HttpEvent<any>) => {
            // custom logic with the response
        }, (err: any) => {
            if (err instanceof HttpErrorResponse) {
                // custom logic with the error (based on status on our case)
            }
        });
    }
}
```

The interceptors have to be added to the `providers` array of the application's module

```javascript
    providers: [
        { provide: HTTP_INTERCEPTORS, useClass: AuthenticatedHttpInterceptor, multi: true }
    ]
```