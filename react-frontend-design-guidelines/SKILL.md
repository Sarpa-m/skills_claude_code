Skill: react-frontend-design-guidelines

This document defines the strict guidelines the agent must follow when developing, refactoring, and designing Front-End components using React. The absolute focus is on creating modern, fluid, scalable, and accessible interfaces, rigorously applying the best Design Patterns in the React ecosystem, especially leveraging Tailwind CSS.

1. UI/UX Principles (Modern and Fluid)

The agent must prioritize the user experience, ensuring the interface is not only functional but also pleasant and responsive.

Fluid Design: Always use smooth transitions for state changes (e.g., hover, focus, opening modals). The design should feel natural and responsive to touch/click.

Mobile-First Responsiveness: The interface must adapt perfectly to any screen size using relative units (rem, vh, vw, %) and modern utilities (Tailwind CSS is mandatory).

Visual Feedback: Always provide immediate feedback for user actions (loading, success, error, and disabled states).

Accessibility (a11y): Use semantic HTML tags and appropriate ARIA attributes. Ensure efficient keyboard navigation.

2. Design Patterns and React Architecture

The code must be highly modular, testable, and maintainable.

2.1. Separation of Concerns (Custom Hooks)

Avoid massive components. Isolate business logic, API calls, and complex state management into Custom Hooks. The component should be primarily concerned with rendering (UI).

2.2. Component Composition

Avoid "Prop Drilling" (excessive property passing). Use the Composition pattern (receiving children or specific slots) to create reusable and flexible components.

2.3. Strict Typing (TypeScript)

All React code must be strongly typed. Interfaces or Types must be defined for all props, states, and function returns. NEVER use any.

3. Tailwind CSS Best Practices

Tailwind CSS is the standard for styling. The agent must adhere strictly to these patterns:

Reuse Patterns: Always look for ways to reuse common utility combinations. Do not repeat long strings of classes if a reusable component or a specific pattern can be applied.

Component Abstraction: When styling becomes too complex in the markup, abstract it thoughtfully within the component itself or through a reusable UI library component.

Consistent Theming: Adhere to the established design tokens (colors, spacing, typography) defined in the Tailwind configuration. Do not hardcode arbitrary values (e.g., use text-blue-600 instead of text-[#2563eb]).

4. Documentation and Comments

Language: English.

Objectivity: Comment on the "why" of logical decisions or specific business rules, not what React or Tailwind natively does.

JSDoc/TSDoc: Use JSDoc to document shared components, describing their main properties.

5. Expected Structure Example

Below is the expected standard for creating a modern component and its associated hook, using Tailwind CSS:

// useUserForm.ts
import { useState, useCallback } from 'react';

/**
 * Custom hook to manage the logic and state of the user form.
 * Isolates the business rule from the view layer.
 */
export function useUserForm(userId?: string) {
  const [isLoading, setIsLoading] = useState(false);

  const saveUser = useCallback(async (data: UserData) => {
    setIsLoading(true);
    try {
      // API persistence logic
    } finally {
      // Ensures the loading state is removed even in case of an error
      setIsLoading(false);
    }
  }, [userId]);

  return { isLoading, saveUser };
}

// SaveButton.tsx
import React from 'react';

interface SaveButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  isLoading?: boolean;
}

/**
 * Primary action button with loading state support and fluid transitions.
 * @param {boolean} isLoading - Disables the button and shows a visual spinner.
 */
export const SaveButton: React.FC<SaveButtonProps> = ({ isLoading, children, ...props }) => {
  return (
    <button
      {...props}
      disabled={isLoading || props.disabled}
      className={`
        px-4 py-2 rounded-lg font-medium text-white transition-all duration-200
        ${isLoading 
          ? 'bg-blue-300 cursor-not-allowed' 
          : 'bg-blue-600 hover:bg-blue-700 active:scale-95'
        }
      `}
    >
      {isLoading ? 'Saving...' : children}
    </button>
  );
};


6. Self-Evaluation Checklist (Agent)

Before finalizing any development or visual refactoring, verify:

[ ] Is the business logic separated from the UI (using Custom Hooks or utilities)?

[ ] Does the component gracefully adapt to small (mobile) and large (desktop) screens?

[ ] Do user interactions (clicks, hovers) have fluid CSS transitions?

[ ] Are Tailwind CSS patterns reused effectively without redundant class strings?

[ ] Is the code strictly typed, without the use of any?

[ ] Do the comments explain complex business rules clearly and in English?

[ ] Crucial Check: Has the code been verified for syntax errors and build readiness before committing? Are all best practices applied?
