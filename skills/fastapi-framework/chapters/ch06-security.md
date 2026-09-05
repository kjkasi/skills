# Chapter 6: Security

## Core Idea
FastAPI provides tools for implementing authentication and authorization using OAuth2, JWT tokens, and password hashing, with automatic integration into interactive documentation.

## Frameworks Introduced
- **OAuth2**: Standard for authentication/authorization. Use for login systems and third-party auth.
- **JWT (JSON Web Tokens)**: Token-based authentication. Use for stateless API authentication.
- **pwdlib**: Password hashing library. Use for secure password storage.
- **PyJWT**: JWT token generation and verification. Use for creating access tokens.

## Key Concepts
- **OAuth2**: Specification for authentication covering multiple flows (password, authorization code, etc.).
- **JWT Token**: Signed token containing user data with expiration. Not encrypted, but tamper-proof.
- **Password Hashing**: Converting passwords to irreversible hashes for secure storage.
- **Security Scopes**: Define required permissions for endpoints (e.g., `admin`, `user`).
- **Bearer Token**: Authentication scheme using `Authorization: Bearer <token>` header.

## Mental Models
- **Use dependencies for authentication**: Create reusable security dependencies for protected endpoints.
- **Think of tokens as temporary credentials**: JWT tokens expire after a set time, requiring re-authentication.
- **Use response models to filter sensitive data**: Never return passwords or secrets in responses.

## Anti-patterns
- **Don't store plaintext passwords**: Always hash passwords before storing them.
- **Don't use weak secret keys**: Generate strong keys with `openssl rand -hex 32`.
- **Don't skip token expiration**: Always set reasonable token lifetimes.

## Code Examples
```python
from datetime import datetime, timedelta, timezone
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

app = FastAPI()

SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["argon2"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

class Token(BaseModel):
    access_token: str
    token_type: str

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password):
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    user = get_user(username=username)
    if user is None:
        raise credentials_exception
    return user

@app.post("/token")
async def login_for_access_token(
    form_data: Annotated[OAuth2PasswordRequestForm, Depends()]
):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}
```

## Reference Tables

| Security Scheme | Use Case | OpenAPI Type |
|----------------|----------|--------------|
| API Key | Simple authentication | `apiKey` |
| OAuth2 Password | Username/password login | `oauth2` |
| OAuth2 Bearer | JWT token authentication | `oauth2` |
| HTTP Basic | Simple HTTP auth | `http` |

## Worked Example
Create a complete JWT authentication flow:
1. User sends credentials to `/token` endpoint
2. Server validates credentials and returns JWT token
3. Client includes token in `Authorization: Bearer <token>` header
4. Server validates token on protected endpoints
5. Token expires after configured time, requiring re-authentication

## Key Takeaways
1. Use OAuth2 with JWT tokens for stateless API authentication.
2. Always hash passwords with a strong algorithm like Argon2.
3. Dependencies can be combined with security scopes for fine-grained authorization.
4. FastAPI automatically documents security schemes in the interactive API docs.

## Connects To
- **Ch 5**: Security is implemented as dependencies using `Depends()`.
- **Ch 7**: Response models should filter sensitive data like passwords.
- **Ch 9**: Deployment requires HTTPS for secure token transmission.
