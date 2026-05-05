Create User
```bash
psql
```

```bash
create role bouncer with login;
```

Set encryption metot
```bash
set password_encryption = 'scram-sha-256';
```
set password
```bash
alter role bouncer with encrypted password 'bouncer2025';
```
create bouncer database
```bash
create database bouncer owner bouncer;
```

```bash
\c bouncer 
```
Create Lookup Function
```bash
create function public.lookup(
inout p_user name,
out p_password text)
returns record language sql security definer set search_path = pg_catalog as
$$select usename,passwd from pg_shadow where usename=p_user$$;
```
Grant Revoke işlemleri
```bash
revoke connect on database bouncer from public;
REVOKE EXECUTE ON FUNCTION public.lookup(name) FROM PUBLIC;
GRANT EXECUTE ON FUNCTION public.lookup(name) TO bouncer;
```