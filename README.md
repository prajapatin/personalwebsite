# Nilesh Prajapati (https://www.nileshprajapati.net)

## Development

### Prerequisites
- Ruby (version 3.3.0)
- Bundler (version 4.0.5)

### Installation

Install dependencies:

```bash
bundle install
```

### Usage

Run the local development server:

```bash
bundle exec jekyll serve
```

The site will be available at `http://localhost:4000`.

### Building

Build the site for production:

```bash
bundle exec jekyll build
```

> **Note:** Always use `bundle exec` to ensure you are using the correct gem versions specified in the `Gemfile` to avoid version mismatches (e.g., with Rake).