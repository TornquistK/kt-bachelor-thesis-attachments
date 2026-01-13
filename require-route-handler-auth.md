```js
/**
 * Rule to ensure Route Handlers have proper authentication checks
 * Targets Next.js App Router API routes (GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD)
 */

const { hasAuthCheck, mergePatterns } = require('../utils/ast-helpers');
const {
  AUTH_PATTERNS,
  ROUTE_HANDLER_METHODS,
  MUTATION_METHODS,
} = require('../utils/patterns');

function isRouteHandlerFile(filename) {
  if (!filename) return false;
  return /route\.(js|ts|jsx|tsx)$/.test(filename);
}

function getExportedFunctionName(node) {
  // export async function GET(request) {}
  if (node.type === 'ExportNamedDeclaration') {
    if (node.declaration && node.declaration.type === 'FunctionDeclaration') {
      return node.declaration.id ? node.declaration.id.name : null;
    }
    // export const GET = async (request) => {}
    if (node.declaration && node.declaration.type === 'VariableDeclaration') {
      const decl = node.declaration.declarations[0];
      if (decl && decl.id && decl.id.type === 'Identifier') {
        return decl.id.name;
      }
    }
  }
  return null;
}

function getFunctionNode(node) {
  if (node.type === 'ExportNamedDeclaration') {
    if (node.declaration && node.declaration.type === 'FunctionDeclaration') {
      return node.declaration;
    }
    if (node.declaration && node.declaration.type === 'VariableDeclaration') {
      const decl = node.declaration.declarations[0];
      if (
        decl &&
        decl.init &&
        (decl.init.type === 'ArrowFunctionExpression' ||
          decl.init.type === 'FunctionExpression')
      ) {
        return decl.init;
      }
    }
  }
  return null;
}

module.exports = {
  meta: {
    type: 'problem',
    docs: {
      description:
        'Ensure Route Handlers have proper authentication checks',
      category: 'Security',
      recommended: true,
    },
    fixable: null,
    messages: {
      missingAuth:
        'Route Handler "{{method}}" is missing authentication check. Add session validation before processing requests.',
      missingAuthMutation:
        'Route Handler "{{method}}" performs mutations but lacks authentication. This could allow unauthorized data modification.',
      recommendAuthPattern:
        'Consider adding: const session = await getServerSession(); if (!session) return new Response("Unauthorized", { status: 401 });',
    },
    schema: [
      {
        type: 'object',
        properties: {
          customAuthPatterns: {
            type: 'array',
            items: {
              type: 'string',
            },
          },
          allowedPublicMethods: {
            type: 'array',
            items: {
              type: 'string',
            },
          },
        },
        additionalProperties: false,
      },
    ],
  },

  create(context) {
    const options = context.options[0] || {};

    // Create local copy with custom patterns merged (no mutation of globals)
    const authPatterns = mergePatterns(AUTH_PATTERNS, options.customAuthPatterns);

    const allowedPublicMethods = new Set(
      (options.allowedPublicMethods || ['GET', 'OPTIONS', 'HEAD']).map((m) =>
        m.toUpperCase()
      )
    );

    const filename = context.getFilename ? context.getFilename() : context.filename;

    if (!isRouteHandlerFile(filename)) {
      return {};
    }

    return {
      ExportNamedDeclaration(node) {
        const methodName = getExportedFunctionName(node);

        if (!methodName || !ROUTE_HANDLER_METHODS.has(methodName)) {
          return;
        }

        const functionNode = getFunctionNode(node);
        if (!functionNode) {
          return;
        }

        const hasAuth = hasAuthCheck(functionNode, authPatterns);
        const isMutation = MUTATION_METHODS.has(methodName);

        if (!hasAuth) {
          if (isMutation) {
            context.report({
              node: functionNode,
              messageId: 'missingAuthMutation',
              data: { method: methodName },
            });
          } else if (!allowedPublicMethods.has(methodName)) {
            context.report({
              node: functionNode,
              messageId: 'missingAuth',
              data: { method: methodName },
            });
          }
        }
      },
    };
  },
};
```