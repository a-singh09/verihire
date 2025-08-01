# Supabase Schema Documentation

This document provides a comprehensive overview of the database schema used in the Trust Match platform.

## Table Overview

The database consists of six main tables that work together to manage candidate verification and trust scoring:

1. `candidates` - Core user data
2. `resumes` - Parsed resume information
3. `verification_requests` - Outgoing verification requests
4. `verifications` - Completed verifications
5. `trust_scores` - AI-generated trust assessments
6. `credentials` - ZK credentials and NFT data

## Table Details

### candidates

Primary table for storing user information.

```sql
CREATE TABLE candidates (
    id TEXT PRIMARY KEY,              -- World ID nullifier hash
    resume_data JSONB DEFAULT '{}',   -- Parsed resume data
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Usage:**

- Stores basic candidate information
- `id` is the World ID nullifier hash
- `resume_data` contains structured resume information
- Includes timestamps for auditing

### resumes

Stores detailed parsed resume information.

```sql
CREATE TABLE resumes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id TEXT NOT NULL REFERENCES candidates(id),
    work_experience JSONB,    -- Array of work experiences
    education JSONB,          -- Array of education details
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Usage:**

- Stores parsed resume data in structured format
- Links to candidate via `user_id`
- JSONB fields allow flexible data structure
- Used for verification matching and AI analysis

### verification_requests

Tracks outgoing verification requests to employers.

```sql
CREATE TABLE verification_requests (
    id SERIAL PRIMARY KEY,
    candidate_id TEXT NOT NULL REFERENCES candidates(id),
    employer_email TEXT NOT NULL,
    company TEXT NOT NULL,
    position TEXT NOT NULL,
    verification_id TEXT UNIQUE NOT NULL,  -- UUID for tracking
    status TEXT NOT NULL DEFAULT 'pending', -- pending/completed/failed
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Usage:**

- Created when sending verification requests
- Tracks status of each verification request
- Links to candidate via `candidate_id`
- `verification_id` used in magic links

### verifications

Stores completed employment verifications.

```sql
CREATE TABLE verifications (
    id TEXT PRIMARY KEY,              -- verification_id from requests
    candidate_id TEXT NOT NULL REFERENCES candidates(id),
    candidate_name TEXT,
    candidate_email TEXT,
    employer_email TEXT NOT NULL,
    company TEXT NOT NULL,
    position TEXT NOT NULL,
    start_date TEXT,
    end_date TEXT,
    verified BOOLEAN NOT NULL,
    employer_world_id TEXT,           -- World ID of verifier
    employer_nullifier TEXT,          -- Prevents duplicate verifications
    verification_proof TEXT,          -- Cryptographic proof
    comments TEXT,
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    status TEXT NOT NULL DEFAULT 'completed'
);
```

**Usage:**

- Stores completed verification results
- Links to candidate via `candidate_id`
- Includes employer World ID for verification
- Contains employment details and dates
- Stores verification proofs

### trust_scores

Stores AI-generated trust scores and analysis.

```sql
CREATE TABLE trust_scores (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    candidate_id TEXT NOT NULL UNIQUE REFERENCES candidates(id),
    score INTEGER NOT NULL CHECK (score >= 0 AND score <= 100),
    analysis TEXT NOT NULL,           -- Detailed AI analysis
    breakdown JSONB DEFAULT '{}',     -- Score breakdown by category
    recommendations TEXT[] DEFAULT '{}',
    risk_factors TEXT[] DEFAULT '{}',
    strengths TEXT[] DEFAULT '{}',
    calculated_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Usage:**

- Generated by ASI AI analysis
- Provides trust score (0-100)
- Includes detailed analysis and breakdowns
- Stores recommendations and risk factors
- Updated periodically as new verifications arrive

### credentials

Stores ZK credentials and NFT data.

```sql
CREATE TABLE credentials (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    candidate_id TEXT NOT NULL REFERENCES candidates(id),
    credential_hash TEXT NOT NULL,    -- Hash of credential data
    zk_proof TEXT,                    -- Zero-knowledge proof
    trust_score INTEGER NOT NULL,
    verification_count INTEGER NOT NULL,
    issuance_date TIMESTAMPTZ NOT NULL,
    status TEXT NOT NULL DEFAULT 'active',
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Usage:**

- Stores credential NFTs
- Contains ZK proofs for privacy
- Links to candidate via `candidate_id`
- Tracks credential status and metadata

## Indexes

The following indexes are created for performance optimization:

```sql
CREATE INDEX idx_candidates_created_at ON candidates(created_at);
CREATE INDEX idx_resumes_user_id ON resumes(user_id);
CREATE INDEX idx_verification_requests_candidate_id ON verification_requests(candidate_id);
CREATE INDEX idx_verification_requests_verification_id ON verification_requests(verification_id);
CREATE INDEX idx_verifications_candidate_id ON verifications(candidate_id);
CREATE INDEX idx_trust_scores_candidate_id ON trust_scores(candidate_id);
CREATE INDEX idx_trust_scores_score ON trust_scores(score);
CREATE INDEX idx_trust_scores_calculated_at ON trust_scores(calculated_at);
CREATE INDEX idx_credentials_candidate_id ON credentials(candidate_id);
```

## Row Level Security (RLS)

All tables have RLS enabled with appropriate policies:

```sql
-- Enable RLS
ALTER TABLE candidates ENABLE ROW LEVEL SECURITY;
ALTER TABLE resumes ENABLE ROW LEVEL SECURITY;
ALTER TABLE verification_requests ENABLE ROW LEVEL SECURITY;
ALTER TABLE verifications ENABLE ROW LEVEL SECURITY;
ALTER TABLE trust_scores ENABLE ROW LEVEL SECURITY;
ALTER TABLE credentials ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Users can manage their own candidate data" ON candidates
    FOR ALL USING (true);

CREATE POLICY "Users can manage their own resumes" ON resumes
    FOR ALL USING (true);

CREATE POLICY "Users can manage their own verification requests" ON verification_requests
    FOR ALL USING (true);

CREATE POLICY "Users can read their own verifications" ON verifications
    FOR SELECT USING (true);

CREATE POLICY "Users can read their own trust scores" ON trust_scores
    FOR SELECT USING (true);

CREATE POLICY "Service role can manage trust scores" ON trust_scores
    FOR ALL USING (auth.role() = 'service_role');

CREATE POLICY "Users can manage their own credentials" ON credentials
    FOR ALL USING (true);
```

## Triggers

Automatic timestamp updates are handled by triggers:

```sql
-- Update timestamps
CREATE TRIGGER update_candidates_updated_at
    BEFORE UPDATE ON candidates
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_trust_scores_updated_at
    BEFORE UPDATE ON trust_scores
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

## Data Flow

1. User signs up → `candidates` record created
2. Resume uploaded → Parsed into `resumes`
3. Verification requested → `verification_requests` created
4. Employer verifies → `verifications` record created
5. AI analyzes → `trust_scores` updated
6. Credential issued → `credentials` record created

## Security Notes

1. All tables use RLS for data isolation
2. Service role required for trust score updates
3. World ID used for verification uniqueness
4. ZK proofs protect sensitive data
5. Employer nullifier prevents duplicate verifications
