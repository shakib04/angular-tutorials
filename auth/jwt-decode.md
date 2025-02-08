I'll show you how to decode and extract data from a JWT token after authentication.

First, create a JWT service:

```typescript
// services/jwt.service.ts
import { Injectable } from '@angular/core';
import { jwtDecode } from 'jwt-decode';

@Injectable({
  providedIn: 'root'
})
export class JwtService {
  decodeToken(token: string): any {
    try {
      return jwtDecode(token);
    } catch(Error) {
      return null;
    }
  }

  getTokenExpirationDate(token: string): Date | null {
    const decoded = this.decodeToken(token);
    
    if (!decoded || !decoded.exp) {
      return null;
    }

    const date = new Date(0);
    date.setUTCSeconds(decoded.exp);
    return date;
  }

  isTokenExpired(token: string): boolean {
    const date = this.getTokenExpirationDate(token);
    return date ? date.valueOf() < new Date().valueOf() : true;
  }
}
```

Then use it in your auth service:

```typescript
// services/auth.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { tap } from 'rxjs/operators';
import { JwtService } from './jwt.service';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private readonly TOKEN_KEY = 'auth_token';
  private userData: any = null;

  constructor(
    private http: HttpClient,
    private jwtService: JwtService
  ) {
    // Load user data from token if exists
    const token = this.getToken();
    if (token) {
      this.userData = this.jwtService.decodeToken(token);
    }
  }

  login(credentials: any) {
    return this.http.post<any>('/api/auth/authenticate', credentials).pipe(
      tap(response => {
        const token = response.token;
        localStorage.setItem(this.TOKEN_KEY, token);
        this.userData = this.jwtService.decodeToken(token);
        localStorage.setItem("role", <string>this.getUserRole());
      })
    );
  }

  getToken(): string | null {
    return localStorage.getItem(this.TOKEN_KEY);
  }

  getUserData() {
    return this.userData;
  }

  // Get specific claims from token
  getUserId(): string | null {
    return this.userData?.sub || null;
  }

  getUserRole(): string | null {
    return this.userData?.role || null;
  }

  getUsername(): string | null {
    return this.userData?.username || null;
  }
}
```

First, install the jwt-decode package:
```bash
npm install jwt-decode
```

Usage in a component:
```typescript
// login.component.ts
@Component({
  // ... component metadata
})
export class LoginComponent {
  constructor(private authService: AuthService) {}

  onSubmit() {
    this.authService.login(this.loginForm.value).subscribe({
      next: () => {
        // Access user data after login
        const userData = this.authService.getUserData();
        console.log('User ID:', this.authService.getUserId());
        console.log('User Role:', this.authService.getUserRole());
        console.log('Username:', this.authService.getUsername());
      },
      error: (error) => {
        console.error('Login failed:', error);
      }
    });
  }
}
```

Key features:
1. Secure token decoding
2. Error handling
3. Expiration checking
4. Easy access to claims
5. Type-safe data access

Let me know if you need any clarification!
