# Deployment Guide for AI Marketing Pipeline

## Overview

This guide covers deployment of the AI-powered marketing pipeline for `rawr_`. The application uses Supabase for database, Vercel for deployment, and requires API keys for various services.

## Prerequisites

### Required Services

1. **Supabase**
   - Database hosting
   - Authentication (optional)
   - Storage (optional)

2. **Vercel** (or similar)
   - Frontend deployment
   - API routes
   - Environment variables

3. **Anthropic API**
   - Claude API access
   - API key for LLM calls

4. **Social Media APIs** (Optional)
   - Twitter API v2
   - LinkedIn API
   - Reddit API
   - Others as needed

## Step 1: Set Up Supabase

### 1.1 Create Supabase Project
1. Go to [supabase.com](https://supabase.com)
2. Create a new project
3. Set up database and authentication

### 1.2 Set Up Database
Run the schema SQL:
```sql
-- Create tables
CREATE TABLE pipeline_runs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL,
  started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  status VARCHAR(20) NOT NULL DEFAULT 'running' CHECK (status IN ('running', 'complete', 'failed')),
  stage VARCHAR(50) NOT NULL DEFAULT 'initialization'
);

CREATE TABLE agent_outputs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id UUID NOT NULL REFERENCES pipeline_runs(id) ON DELETE CASCADE,
  agent_name VARCHAR(50) NOT NULL,
  content JSONB NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'pending_approval' CHECK (status IN ('draft', 'pending_approval', 'approved', 'rejected', 'published')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE approvals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_output_id UUID NOT NULL REFERENCES agent_outputs(id) ON DELETE CASCADE,
  approved_by TEXT NOT NULL,
  approved_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  decision VARCHAR(10) NOT NULL CHECK (decision IN ('approved', 'rejected'))
);

CREATE TABLE pipeline_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  run_id UUID NOT NULL REFERENCES pipeline_runs(id) ON DELETE CASCADE,
  message TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE company_context (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL,
  brand_voice TEXT,
  past_content JSONB,
  competitors JSONB,
  target_keywords JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 1.3 Get Supabase Credentials
Copy these values:
- Project URL
- Anon key
- Service role key (for server-side operations)

## Step 2: Set Up Vercel

### 2.1 Create Vercel Project
1. Go to [vercel.com](https://vercel.com)
2. Create new project
3. Connect your Git repository

### 2.2 Configure Environment Variables
Add these environment variables to your Vercel project:
```bash
# Supabase
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
VITE_SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key

# Anthropic
VITE_ANTHROPIC_API_KEY=your_anthropic_api_key

# Optional: Social Media APIs
VITE_OPENAI_API_KEY=your_openai_api_key
VITE_GOOGLE_API_KEY=your_google_api_key
VITE_REDDIT_CLIENT_ID=your_reddit_client_id
VITE_REDDIT_CLIENT_SECRET=your_reddit_client_secret
VITE_TWITTER_API_KEY=your_twitter_api_key
VITE_TWITTER_API_SECRET=your_twitter_api_secret
VITE_LINKEDIN_CLIENT_ID=your_linkedin_client_id
VITE_LINKEDIN_CLIENT_SECRET=your_linkedin_client_secret

# Application Settings
VITE_APP_NAME=rawr_ AI CMO
VITE_APP_URL=https://yourwebsite.com
VITE_APP_EMAIL=hello@yourwebsite.com
```

### 2.3 Deploy
```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel --prod
```

## Step 3: Set Up Anthropic API

### 3.1 Get Anthropic API Key
1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Create an account
3. Generate API key
4. Copy the key

### 3.2 Configure Environment Variable
Add to your Vercel environment variables:
```bash
VITE_ANTHROPIC_API_KEY=sk-ant-your-key-here
```

## Step 4: Configure Social Media APIs (Optional)

### 4.1 Twitter API v2
1. Create Twitter Developer account
2. Create app and get credentials
3. Configure OAuth 2.0
4. Add environment variables:
```bash
VITE_TWITTER_API_KEY=your_twitter_api_key
VITE_TWITTER_API_SECRET=your_twitter_api_secret
```

### 4.2 LinkedIn API
1. Create LinkedIn Developer account
2. Create app and get credentials
3. Configure OAuth 2.0
4. Add environment variables:
```bash
VITE_LINKEDIN_CLIENT_ID=your_linkedin_client_id
VITE_LINKEDIN_CLIENT_SECRET=your_linkedin_client_secret
```

### 4.3 Reddit API
1. Create Reddit Developer account
2. Create app and get credentials
3. Configure OAuth 2.0
4. Add environment variables:
```bash
VITE_REDDIT_CLIENT_ID=your_reddit_client_id
VITE_REDDIT_CLIENT_SECRET=your_reddit_client_secret
```

## Step 5: Seed Database

### 5.1 Create Seed Script
Create `src/supabase/seed.ts`:
```typescript
import { supabaseAdmin } from '@/utils/supabase-admin';

export async function seedCompanyContext() {
  const { data, error } = await supabaseAdmin
    .from('company_context')
    .upsert([{
      company_id: 'test-company',
      name: 'rawr_',
      description: 'AI CMO platform for automated SEO, GEO optimization, and content marketing',
      brand_voice: 'Technical but accessible, data-driven, action-oriented',
      past_content: [
        'Blog post on AI trends',
        'Case study on SEO tools',
        'Guide to content automation'
      ],
      competitors: [
        'SEMrush',
        'Ahrefs',
        'Moz',
        'SurferSEO',
        'Frase'
      ],
      target_keywords: [
        'AI SEO automation',
        'GEO optimization',
        'content marketing AI',
        'local citation management',
        'SEO audit tools'
      ]
    }], { onConflict: 'company_id' });

  if (error) {
    console.error('Error seeding company context:', error);
  } else {
    console.log('Company context seeded successfully');
  }
}
```

### 5.2 Run Seed Script
```bash
npm run seed
```

## Step 6: Test Deployment

### 6.1 Local Testing
```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Run tests
npm test
```

### 6.2 Production Testing
1. Visit your deployed application
2. Test the pipeline functionality
3. Verify approval workflow
4. Check distribution queue

## Step 7: Monitor and Maintain

### 7.1 Monitor Logs
- Check Vercel logs for errors
- Monitor Supabase logs
- Track API usage

### 7.2 Set Up Monitoring
- Set up error tracking (Sentry, etc.)
- Monitor API response times
- Track usage metrics

### 7.3 Regular Maintenance
- Update dependencies
- Monitor API key usage
- Clean up old data
- Update social media API credentials

## Troubleshooting

### Common Issues

#### API Key Errors
```bash
# Error: Missing Anthropic API key
# Solution: Check VITE_ANTHROPIC_API_KEY environment variable
```

#### Database Connection Errors
```bash
# Error: Connection to Supabase failed
# Solution: Check VITE_SUPABASE_URL and VITE_SUPABASE_ANON_KEY
```

#### CORS Errors
```bash
# Error: CORS policy violation
# Solution: Configure CORS in Vercel or Supabase
```

#### Rate Limiting
```bash
# Error: API rate limit exceeded
# Solution: Implement rate limiting or upgrade API plans
```

## Support

### Getting Help
1. Check the documentation
2. Review the code
3. Test locally
4. Check logs
5. Contact support if needed

### Reporting Issues
1. Create an issue in the repository
2. Include error messages
3. Provide steps to reproduce
4. Include logs if available

## Conclusion

This deployment guide covers the setup and deployment of the AI-powered marketing pipeline. The application is now ready for production use with real AI generation, mandatory approval workflow, and multi-channel distribution capabilities.

The pipeline provides a robust foundation for automated content marketing with quality control through human approval and real-time monitoring capabilities.