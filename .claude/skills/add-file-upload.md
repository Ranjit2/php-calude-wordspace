# Skill: Add File Upload

> **Laravel:** `Storage::disk('private')` + `UploadedFile` + `Storage::fake()` in tests
> **Symfony:** `Request->files->get()` + Flysystem adapter
> **Core PHP:** `$_FILES` + `move_uploaded_file()` + manual path management


## Security checklist — read before writing a single line
See rules/security.md file upload section. Every point is mandatory.

## Implementation checklist
- [ ] 1.  Validate the upload:
          - **Laravel:** FormRequest rule `'file' => ['required', 'file', 'mimes:pdf,docx,jpg,png', 'max:10240']`
          - **Symfony:** `#[Assert\File(maxSize: '10M', mimeTypes: ['application/pdf'])]`
          - **Core PHP:** check `$_FILES['file']['type']` against an allowlist, validate size
- [ ] 2.  Store with UUID filename on private storage:
          - **Laravel:** `$file->storeAs('uploads', Str::uuid().'.'.$file->extension(), 'private')`
          - **Symfony:** Flysystem private adapter + `Str::uuid()` filename
          - **Core PHP:** `move_uploaded_file()` to a non-web-accessible folder with `uuid` filename
- [ ] 3.  Store the relative path in DB — never the full server path
- [ ] 4.  Generate time-limited access URL:
          - **Laravel:** `Storage::disk('private')->temporaryUrl($path, now()->addMinutes(15))`
          - **Symfony/Core PHP:** generate a signed token, serve file through a controller that checks expiry
- [ ] 5.  Dispatch virus scan job (production): `ScanUploadedFile::dispatch($path)`
- [ ] 6.  For PDFs: dispatch extraction job — never extract synchronously in controller
- [ ] 7.  Write upload test:
          - **Laravel:** `Storage::fake('private')` + `UploadedFile::fake()->create('file.pdf', 500, 'application/pdf')`
          - **Symfony:** `UploadedFile` from `symfony/http-foundation` + in-memory Flysystem
          - **Core PHP:** mock `$_FILES` or use a test helper

## Storage config (config/filesystems.php)
```php
'disks' => [
    'private' => [
        'driver'     => 'local',
        'root'       => storage_path('app/private'),
        'visibility' => 'private',
    ],
    's3-private' => [  // production
        'driver'     => 's3',
        'key'        => env('AWS_ACCESS_KEY_ID'),
        'secret'     => env('AWS_SECRET_ACCESS_KEY'),
        'region'     => env('AWS_DEFAULT_REGION'),
        'bucket'     => env('AWS_PRIVATE_BUCKET'),
        'visibility' => 'private',
    ],
],
```
