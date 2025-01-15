Config package.json

```yaml
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "A project example",
  "main": "index.js",
  "scripts": {
    "test": "jest",
    "build": "npm run prettier && npm run test && npm run release",
    "prettier": "prettier --write \"src/**/*.js\"",
    "commit": "cz",
    "release": "semantic-release",
    "snyk:auth": "snyk auth",
    "snyk:test": "snyk test --severity-threshold=high",
    "snyk:monitor": "snyk monitor"
  },
  "devDependencies": {
    "prettier": "^3.0.0",
    "semantic-release": "^20.0.0",
    "@semantic-release/commit-analyzer": "^10.0.0",
    "@semantic-release/release-notes-generator": "^11.0.0",
    "@semantic-release/changelog": "^6.0.0",
    "@semantic-release/git": "^10.0.0",
    "commitizen": "^4.0.0",
    "cz-conventional-changelog": "^3.0.0",
    "jest": "^29.0.0",
    "snyk": "^1.1255.0"
  }
}
```

---

Config .releaserc.json

```yaml
{
  "branches": [
    { "name": "main" },
    { "name": "dev", "prerelease": "alpha" },
    { "name": "staging", "prerelease": "beta" },
    { "name": "release", "prerelease": "rc" }
  ],
  "repositoryUrl": "https://bitbucket.org/your-workspace/your-repo.git",
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    "@semantic-release/github",
    [
      "@semantic-release/git",
      {
        "assets": ["CHANGELOG.md", "package.json", "package-lock.json"],
        "message": "chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
      }
    ]
  ]
}
```

---

Config .prettierrc.json

```yaml
{
  "extends": ["eslint:recommended", "plugin:prettier/recommended"],
  "env": {
	"es6": true,
	"node": true,
	"browser": true
  },
  "rules": {
	"prettier/prettier": [
	  "error",
	  {
	    "useTabs": false
		“printWidth": 80, // Max line length 
		"tabWidth": 2, // Soft tab size of 2 
		"useTabs": false, // Use spaces instead of tabs 
		"semi": false, // No semicolons 
		"singleQuote": true, // Use single quotes 
		"trailingComma": "es5", // Commas for objects and arrays   
		"bracketSpacing": true, // Space between brackets 
		"arrowParens": "always", // Parens around arrow function params
		"endOfLine": "lf"
	  }
	]
  }
}
```