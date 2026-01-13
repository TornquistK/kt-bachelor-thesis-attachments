```ts
import tsParser from '@typescript-eslint/parser';
import nextjsSecurity from 'eslint-plugin-nextjs-security';

const eslintConfig = [
  {
    ignores: [
      '**/node_modules',
      '**/.next',
      '**/dist',
      '**/out',
      '**/build',
      '**/public',
      '**/@generated',
      'packages/database/src/db/types.ts',
      'packages/database/src/@generated/prisma',
      'next-env.d.ts',
      'apps/storybook/storybook-static/**',
      'packages/eslint-nextjs-security/**',
    ],
  },
  {
    files: ['**/*.ts', '**/*.tsx', '**/*.js', '**/*.jsx', '**/*.mjs'],
    languageOptions: {
      parser: tsParser,
      parserOptions: {
        ecmaVersion: 'latest',
        sourceType: 'module',
        ecmaFeatures: {
          jsx: true,
        },
      },
    },
    plugins: {
      'nextjs-security': nextjsSecurity,
    },
    rules: {
      ...nextjsSecurity.configs.recommended.rules,
    },
  },
];

export default eslintConfig;
```