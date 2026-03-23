# Verification Guidelines

## What Constitutes a Factual Claim

A factual claim is a statement that can be objectively verified or disproven through evidence.

### Factual Claims (Verify These)

**Technical specifications:**
- "Claude Sonnet 4.5 has a 200K token context window"
- "Python 3.12 was released in October 2023"
- "React 18 introduced automatic batching"

**Statistics and numbers:**
- "The global population reached 8 billion in 2022"
- "Node.js 20 improves performance by 30%"
- "Bitcoin's market cap exceeded $1 trillion in 2021"

**Historical facts:**
- "The iPhone was first released in 2007"
- "Git was created by Linus Torvalds"
- "The COVID-19 pandemic began in late 2019"

**Attributable statements:**
- "According to the WHO, vaccines save millions of lives"
- "Einstein said 'Imagination is more important than knowledge'"
- "The New York Times reported that..."

**Quantifiable performance claims:**
- "This algorithm runs in O(n log n) time"
- "SQLite is faster than PostgreSQL for read operations"
- "Chrome uses more memory than Firefox"

### Non-Factual Statements (Mark as N/A)

**Opinions and judgments:**
- "React is better than Vue"
- "Python is the best language for beginners"
- "This design looks professional"

**Subjective experiences:**
- "I find TypeScript easier to debug"
- "This API is intuitive to use"
- "The learning curve is steep"

**Future predictions:**
- "AI will replace programmers in 5 years"
- "Quantum computing will revolutionize cryptography"
- "The metaverse will be mainstream by 2030"

**Recommendations without factual basis:**
- "You should use PostgreSQL for this project"
- "Always write unit tests before production"
- "Microservices are worth the complexity"

**Rhetorical questions:**
- "Who doesn't love clean code?"
- "Isn't it obvious that performance matters?"

**Metaphors and analogies:**
- "Code is poetry"
- "Debugging is like detective work"
- "A database is like a filing cabinet"

## Source Quality Standards

### Tier 1: Authoritative Sources (Highest Confidence)

Use these sources for verification whenever possible:

**Official documentation:**
- Product websites and official docs (platform.claude.com, reactjs.org)
- API references and changelogs
- Release notes and announcements
- Government websites (.gov domains)
- Academic institution publications (.edu domains)

**Primary sources:**
- Original research papers
- Official statistics from recognized bodies
- Direct quotes from verified interviews
- Press releases from companies

**Established references:**
- IEEE, ACM, and other professional organizations
- Standards bodies (W3C, IETF, ISO)
- Peer-reviewed journals
- Authoritative encyclopedias (with citations)

### Tier 2: Credible Secondary Sources (Medium Confidence)

Use these when Tier 1 sources are unavailable, but cross-reference when possible:

**Reputable news outlets:**
- Major tech news sites (TechCrunch, The Verge, Ars Technica)
- Established newspapers with tech sections
- Industry publications (IEEE Spectrum, ACM Communications)

**Well-maintained aggregators:**
- MDN Web Docs (Mozilla)
- Stack Overflow Documentation
- GitHub repositories with active maintenance

**Expert blogs:**
- Posts from recognized experts in the field
- Company engineering blogs (with clear authorship)
- Conference presentations and talks

### Tier 3: Use with Caution (Low Confidence)

These sources may contain errors or outdated information:

**Community content:**
- Wikipedia (check the citations, not the article itself)
- Forum posts and discussions
- Blog posts without clear expertise
- Social media posts (even from experts)

**Automated aggregators:**
- Sites that scrape multiple sources
- AI-generated content
- Comparison sites without methodology

### Sources to Avoid

Do NOT use these for verification:

**Unreliable sources:**
- Outdated documentation
- Unofficial wikis without citations
- Anonymous sources
- Sites known for misinformation
- Marketing materials (without fact-checking)

**Circular references:**
- Sources that cite each other
- Copies of the same content
- Mirrors without original source

## Verification Process

### Step 1: Extract the Claim

Identify the specific factual assertion:
- **Full sentence:** "Python 3.12, released in October 2023, includes better error messages"
- **Factual claims to verify:**
  1. Python 3.12 was released in October 2023
  2. Python 3.12 includes better error messages

### Step 2: Construct Search Query

Create specific, unambiguous search queries:

**Good queries:**
- "Python 3.12 release date official"
- "Python 3.12 error message improvements changelog"
- "Claude Sonnet 4.5 context window 2026"

**Poor queries:**
- "Python release" (too vague)
- "better error messages" (no context)
- "latest version" (ambiguous)

**Search tips:**
- Include version numbers and specific terms
- Add "official" or "documentation" for authoritative sources
- Include the current year for recent information
- Use quotes for exact phrases

### Step 3: Evaluate Sources

For each source found:

1. **Check source tier** - Is it authoritative, credible, or questionable?
2. **Verify recency** - Is the information current?
3. **Look for specifics** - Does it directly address the claim?
4. **Check for updates** - Has information been superseded?
5. **Cross-reference** - Do multiple sources agree?

### Step 4: Determine Verification Status

**✅ Verified:**
- Tier 1 source directly confirms the claim
- Multiple Tier 2 sources agree, or
- Tier 2 source + supporting evidence

**⚠️ Partially Accurate:**
- Core claim is correct but details are wrong
- Information is outdated but was accurate
- Claim lacks important context or caveats

**❌ Inaccurate:**
- Sources contradict the claim
- Claim contains factual errors
- Numbers or dates are wrong

**🔍 Unable to Verify:**
- No authoritative sources found
- Sources conflict with each other
- Information is too recent/obscure
- Claim is too vague to verify

## Edge Cases and Special Situations

### Conflicting Sources

When authoritative sources disagree:
1. Note the discrepancy in the verification notes
2. Cite both sources with their claims
3. Prefer the most recent information
4. Consider if both could be correct in different contexts

**Example:**
- Source A (2023): "Feature X is in beta"
- Source B (2024): "Feature X is generally available"
- **Resolution:** Both are correct for their respective dates; use the most recent

### Rapidly Changing Information

For fast-moving topics (AI models, software versions, current events):
1. Prioritize recent sources (within 3-6 months)
2. Check for official announcements
3. Note the verification date in your output
4. Flag that information may change

### Citation Verification

When the blog post quotes or cites sources:
1. Find the original source
2. Verify the quote/claim is accurate
3. Check if the source is being used in proper context
4. Note if the link is broken or outdated

**Example:**
- **Blog claim:** "According to the WHO, COVID-19 vaccines are 95% effective"
- **Verification steps:**
  1. Find the WHO statement
  2. Check if 95% is accurate (may vary by vaccine)
  3. Verify this applies to the right context (original variant vs. current)
  4. Note any important caveats WHO provided

### Statistical Claims

For numbers and statistics:
1. Find the original source of the statistic
2. Verify the number is quoted correctly
3. Check the date and context
4. Note if it's an estimate, average, or precise figure

### Technical Claims

For code, algorithms, or technical specifications:
1. Check official documentation
2. Test if verifiable (e.g., code samples)
3. Look for benchmark sources for performance claims
4. Note version-specific behavior

## Quality Checklist

Before marking a claim as verified:

- [ ] Source is Tier 1 or 2 quality
- [ ] Source directly addresses the specific claim
- [ ] Information is current (or appropriately dated)
- [ ] URL is accessible and valid
- [ ] Source page contains the verification details
- [ ] No conflicting information from equally authoritative sources
- [ ] Context is appropriate for the claim
