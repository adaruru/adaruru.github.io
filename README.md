# Amanda's Blog

Personal blog built with [Hugo](https://gohugo.io/) and [Toha](https://github.com/hugo-toha/toha) theme.

## Prerequisites

- [Hugo Extended](https://gohugo.io/installation/) (v0.148.2 or later)
- [Go](https://golang.org/dl/) (for Hugo modules)
- [Node.js](https://nodejs.org/) (for npm packages)

## Local Development

### Quick Start (Make)

```bash
make run
```

This will:
1. Pack Hugo modules npm dependencies
2. Install npm packages
3. Open browser at http://localhost:1313
4. Start Hugo server with live reload

### Manual Steps

# install 

```shell
choco install -y golang
choco install -y hugo-extended

hugo version
go version
```

```bash
# 1. Pack npm dependencies from Hugo modules
hugo mod npm pack

# 2. Install npm packages
npm install

# 3. Start development server
hugo server --disableFastRender --watch
```

Visit http://localhost:1313 to view the site.

### Build for Production

```bash
hugo --minify
```

Output will be in the `public/` directory.

## Blog

```
.
├── content/          # Site content (posts, notes, pages)
│   ├── posts/        # Blog posts
│   └── notes/        # Technical notes (Go, Bash, C#, etc.)
```
