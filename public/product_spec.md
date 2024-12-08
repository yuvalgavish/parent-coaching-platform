# Parent Coaching Platform

## Complete MVP Specification

## Table of Contents

1. [Product Overview](#1-product-overview)  
2. [Core Functionality](#2-core-functionality)  
3. [User Experiences](#3-user-experiences)  
4. [Technical Architecture](#4-technical-architecture)  
5. [Database Schema](#5-database-schema)  
6. [Critical Flows](#6-critical-flows)  
7. [Implementation Examples](#7-implementation-examples)  
8. [API Specifications](#8-api-specifications)  
9. [Error Handling](#9-error-handling)  
10. [Business Logic & Limitations](#10-business-logic--limitations)

## 1\. Product Overview

### 1.1 Purpose

A dual-layer SaaS platform enabling:

- Parent coaches to create structured event tracking systems for clients  
- Parents to document specific parenting situations in real-time

### 1.2 MVP Scope

Core features only:

- Basic account management  
- Template creation and assignment  
- Real-time event logging  
- Essential progress tracking  
- Basic coach-parent communication

### 1.3 MVP Limitations

- Basic subscription model only  
- Limited customization options  
- English language only  
- Basic analytics only  
- Web platform only (responsive)

## 2\. Core Functionality

### 2.1 Coach Layer

Essential features:

- Account and profile management  
- Parent client management  
- Event template creation  
- Parent progress monitoring  
- Basic subscription handling

### 2.2 Parent Layer

Essential features:

- Simple account access  
- Real-time event logging  
- Progress history viewing  
- Basic coach communication

## 3\. User Experiences

### 3.1 Coach Experience

#### Account & Dashboard

- Professional profile creation  
- Client list management  
- Template management overview  
- Basic analytics view

#### Template Creation

- Event type definition  
- Step-by-step configuration  
- Response type selection  
- Basic template preview

#### Client Management

- Send parent invitations  
- Monitor event logging  
- View progress data  
- Basic communication

### 3.2 Parent Experience

#### Event Logging

- Select event type  
- Step-by-step guided logging  
- Real-time submission  
- View submission history

#### Communication

- View coach messages  
- Send basic updates  
- See basic progress

## 4\. Technical Architecture

### 4.1 Technology Stack

- Frontend: Vite + React  
- Backend: Supabase  
- Database: PostgreSQL  
- Authentication: Supabase Auth  
- Payments: Stripe  
- Hosting: Vercel (frontend), Supabase (backend)

### 4.2 System Components

graph TD

    A\[Client Application\] \--\>|API Requests| B\[Supabase Backend\]

    B \--\> C\[(PostgreSQL Database)\]

    B \--\> D\[Authentication\]

    B \--\> E\[Real-time Updates\]

    A \--\>|Static Assets| F\[Vercel CDN\]

    A \--\>|Payments| G\[Stripe\]

### 4.3 State Management

interface AppState {

  // Authentication

  auth: {

    user: User | null;

    isLoading: boolean;

    error: Error | null;

  };

  

  // Coach State

  coach?: {

    templates: Template\[\];

    parents: Parent\[\];

    currentTemplate: Template | null;

  };

  

  // Parent State

  parent?: {

    templates: Template\[\];

    currentEvent: EventLog | null;

    history: EventLog\[\];

  };

}

## 5\. Database Schema

### 5.1 Core Tables

\-- User Management

CREATE TABLE profiles (

    id UUID REFERENCES auth.users PRIMARY KEY,

    email TEXT UNIQUE NOT NULL,

    full\_name TEXT,

    user\_type TEXT NOT NULL CHECK (user\_type IN ('coach', 'parent')),

    created\_at TIMESTAMPTZ DEFAULT now()

);

CREATE TABLE coach\_profiles (

    id UUID REFERENCES profiles(id) PRIMARY KEY,

    business\_name TEXT,

    stripe\_customer\_id TEXT,

    subscription\_status TEXT,

    subscription\_tier TEXT DEFAULT 'basic',

    max\_parents INTEGER DEFAULT 10

);

\-- Coach-Parent Relationships

CREATE TABLE coach\_parent\_connections (

    id UUID PRIMARY KEY DEFAULT uuid\_generate\_v4(),

    coach\_id UUID REFERENCES coach\_profiles(id),

    parent\_id UUID REFERENCES profiles(id),

    status TEXT DEFAULT 'pending',

    created\_at TIMESTAMPTZ DEFAULT now(),

    UNIQUE(coach\_id, parent\_id)

);

\-- Event Templates

CREATE TABLE event\_templates (

    id UUID PRIMARY KEY DEFAULT uuid\_generate\_v4(),

    coach\_id UUID REFERENCES coach\_profiles(id),

    name TEXT NOT NULL,

    created\_at TIMESTAMPTZ DEFAULT now()

);

CREATE TABLE template\_steps (

    id UUID PRIMARY KEY DEFAULT uuid\_generate\_v4(),

    template\_id UUID REFERENCES event\_templates(id) ON DELETE CASCADE,

    step\_order INTEGER NOT NULL,

    step\_type TEXT NOT NULL,

    question TEXT NOT NULL,

    allow\_multiple BOOLEAN DEFAULT false

);

CREATE TABLE step\_options (

    id UUID PRIMARY KEY DEFAULT uuid\_generate\_v4(),

    step\_id UUID REFERENCES template\_steps(id) ON DELETE CASCADE,

    option\_text TEXT NOT NULL,

    option\_order INTEGER NOT NULL

);

\-- Event Logging

CREATE TABLE event\_logs (

    id UUID PRIMARY KEY DEFAULT uuid\_generate\_v4(),

    parent\_id UUID REFERENCES profiles(id),

    template\_id UUID REFERENCES event\_templates(id),

    logged\_at TIMESTAMPTZ DEFAULT now()

);

CREATE TABLE event\_responses (

    id UUID PRIMARY KEY DEFAULT uuid\_generate\_v4(),

    event\_log\_id UUID REFERENCES event\_logs(id) ON DELETE CASCADE,

    step\_id UUID REFERENCES template\_steps(id),

    response\_data JSONB NOT NULL,

    created\_at TIMESTAMPTZ DEFAULT now()

);

## 6\. Critical Flows

### 6.1 Parent Invitation Flow

sequenceDiagram

    Coach-\>\>System: Enter parent's email

    System-\>\>Parent: Send invitation email

    Parent-\>\>System: Click invitation link

    System-\>\>Parent: Show registration

    Parent-\>\>System: Complete registration

    System-\>\>Coach: Notify parent joined

    System-\>\>Parent: Show templates

### 6.2 Template Creation Flow

sequenceDiagram

    Coach-\>\>System: Create new template

    Coach-\>\>System: Add template name

    loop Add Steps

        Coach-\>\>System: Select step type

        Coach-\>\>System: Add question

        Coach-\>\>System: Define options

    end

    Coach-\>\>System: Save template

    System-\>\>Coach: Confirm creation

### 6.3 Event Logging Flow

sequenceDiagram

    Parent-\>\>System: Select event type

    loop For Each Step

        System-\>\>Parent: Show question

        Parent-\>\>System: Submit response

        System-\>\>System: Save response

    end

    System-\>\>Parent: Confirm completion

    System-\>\>Coach: Update dashboard

## 7\. Implementation Examples

### 7.1 Template Example

const bedtimeTemplate \= {

  name: "Bedtime Routine",

  steps: \[

    {

      id: "step1",

      type: "action",

      question: "Which approach did you use tonight?",

      options: \[

        "Current: Extended negotiation",

        "Recommended: Clear boundaries"

      \]

    },

    {

      id: "step2",

      type: "duration",

      question: "How long did bedtime take?",

      options: \[

        "5-10 minutes",

        "10-15 minutes",

        "15-30 minutes",

        "Over 30 minutes"

      \]

    },

    {

      id: "step3",

      type: "emotions",

      question: "What emotions did your child express?",

      options: \["Calm", "Resistant", "Cooperative"\],

      allowMultiple: true

    }

  \]

};

### 7.2 Event Log Example

const eventLog \= {

  templateId: "bedtime-routine",

  timestamp: "2024-12-08T20:30:00Z",

  responses: \[

    {

      stepId: "step1",

      value: "Recommended: Clear boundaries"

    },

    {

      stepId: "step2",

      value: "10-15 minutes"

    },

    {

      stepId: "step3",

      value: \["Calm", "Cooperative"\]

    }

  \]

};

## 8\. API Specifications

### 8.1 Core Endpoints

interface CoreEndpoints {

    // Authentication

    '/auth/signup': {

        POST: {

            body: {

                email: string;

                password: string;

                userType: 'coach' | 'parent';

            };

        };

    };

    // Coach Endpoints

    '/api/coach/templates': {

        GET: { response: Template\[\] };

        POST: {

            body: {

                name: string;

                steps: TemplateStep\[\];

            };

        };

    };

    // Parent Endpoints

    '/api/parent/events': {

        GET: { response: EventTemplate\[\] };

        POST: {

            body: {

                templateId: string;

                responses: EventResponse\[\];

            };

        };

    };

}

## 9\. Error Handling

### 9.1 Critical Errors

const criticalErrors \= {

    AUTH: {

        INVALID\_CREDENTIALS: "Invalid email or password",

        EXPIRED\_INVITATION: "Invitation has expired",

        EMAIL\_IN\_USE: "Email already registered"

    },

    TEMPLATE: {

        INVALID\_STRUCTURE: "Template structure is invalid",

        MISSING\_REQUIRED: "Required fields are missing"

    },

    EVENT: {

        SAVE\_FAILED: "Failed to save event",

        INVALID\_RESPONSE: "Invalid response format"

    }

};

### 9.2 Error Recovery

const handleError \= (error: Error) \=\> {

    console.error(error);

    if (isNetworkError(error)) {

        return "Connection lost. Please check your internet.";

    }

    if (isAuthError(error)) {

        return "Please sign in again.";

    }

    return "An error occurred. Please try again.";

};

## 10\. Business Logic & Limitations

### 10.1 MVP Constraints

- Maximum 10 active parents per coach  
- Maximum 10 templates per coach  
- Maximum 5 steps per template  
- Basic analytics only  
- Event logging within last 24 hours only

### 10.2 Success Metrics

1. Usage Metrics:  
     
   - Active coaches  
   - Active parents  
   - Events logged per day  
   - Template usage

   

2. Performance Metrics:  
     
   - System uptime  
   - Response times  
   - Error rates

### 10.3 Future Considerations

Features intentionally left for future versions:

- Template sharing  
- Advanced analytics  
- Mobile applications  
- Multiple languages  
- Custom branding
