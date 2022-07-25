---
sidebar_label: Using Route Guards
sidebar_position: 8
---

# Using Route Guards

> Guards should only be applied to protected URLs. There should be no guard active on the default route where the authorization request is processed.

Please refer to the auto-login guard in this repo as a reference. It is important that the callback logic can be run on a route without the guard running or run before the guard logic.

```ts
import { Injectable } from '@angular/core';
import { ActivatedRouteSnapshot, CanActivate, Router, RouterStateSnapshot, UrlTree } from '@angular/router';
import { OidcSecurityService } from 'angular-auth-oidc-client';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable({ providedIn: 'root' })
export class AuthorizationGuard implements CanActivate {
  constructor(private oidcSecurityService: OidcSecurityService, private router: Router) {}

  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean | UrlTree> {
    return this.oidcSecurityService.isAuthenticated$.pipe(
      map(({ isAuthenticated }) => {
        // allow navigation if authenticated
        if (isAuthenticated) {
          return true;
        }

        // redirect if not authenticated
        return this.router.parseUrl('/unauthorized');
      })
    );
  }
}
```

To apply the guard for specific routes you have to add it to the route configuration e.g. with `canActivate`:

```ts
const appRoutes: Routes = [
  {
    path: 'protected',
    component: <yourComponent>,
    canActivate: [AuthorizationGuard]
  },
  // ...
];
```

All other guard types like `canLoad` or `canActivateChild` work in a similar way. However, the guard class has to implement the respective interfaces and methods accordingly.
