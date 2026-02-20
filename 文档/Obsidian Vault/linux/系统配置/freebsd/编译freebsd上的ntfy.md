```yaml
  -

    id: ntfy_freebsd_amd64

    binary: ntfy

    env:

      - CGO_ENABLED=0

    tags: [noserver]

    ldflags:

      - "-X main.version={{.Version}} -X main.commit={{.Commit}} -X main.date={{.Date}}"

    goos: [freebsd]

    goarch: [amd64]
	```