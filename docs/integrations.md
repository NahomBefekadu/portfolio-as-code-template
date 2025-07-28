# Integration Guide

This guide covers how to integrate your Portfolio as Code system with other tools and workflows to maximize productivity and automate routine tasks.

## Database Integration: Codex Application

The [Codex application](https://github.com/example/codex-app) provides a rich web interface for your portfolio content.

### Setup Process

1. **Clone and Install**
```bash
git clone https://github.com/example/codex-app.git
cd codex-app
npm install
```

2. **Configure Database**
```bash
# Copy environment template
cp .env.example .env

# Edit database connection
DATABASE_URL="postgresql://username:password@localhost:5432/codex"
```

3. **Set Up Database**
```bash
npx prisma migrate dev
npx prisma generate
```

4. **Configure Portfolio Sync**
```json
// codex.config.json
{
  "sync": {
    "enabled": true,
    "direction": "bidirectional",
    "portfolioPath": "/path/to/your/portfolio"
  }
}
```

5. **Initial Sync**
```bash
npm run sync:import
npm run dev
```

### Workflow Integration

**Daily Workflow:**
1. Use Codex web interface for quick content creation
2. Rich markdown editor with live preview
3. Full-text search across all content
4. Tag-based organization and filtering

**Periodic Sync:**
```bash
# Pull changes from files to database
npm run sync:import

# Push database changes to files  
npm run sync:export

# Bidirectional sync (handles conflicts)
npm run sync:bidirectional
```

**Benefits:**
- Rich editing experience
- Powerful search capabilities
- Analytics and insights
- Export functionality
- Mobile-friendly interface

## Static Site Integration

Transform your portfolio into a public website using static site generators.

### Next.js Integration

1. **Set Up Next.js Site**
```bash
npx create-next-app@latest portfolio-site --typescript --tailwind --app
cd portfolio-site
```

2. **Install Dependencies**
```bash
npm install gray-matter remark remark-html remark-gfm
npm install @types/react @types/node
```

3. **Content Parsing Utility**
```typescript
// lib/portfolio.ts
import fs from 'fs'
import path from 'path'
import matter from 'gray-matter'
import { remark } from 'remark'
import html from 'remark-html'
import remarkGfm from 'remark-gfm'

const portfolioDirectory = path.join(process.cwd(), '../portfolio-repo/content')

export interface ContentItem {
  id: string
  title: string
  date: string
  tags: string[]
  category: string
  content: string
  [key: string]: any
}

export function getAllContent(type: 'learnings' | 'achievements' | 'blog'): ContentItem[] {
  const contentDir = path.join(portfolioDirectory, type)
  const years = fs.readdirSync(contentDir)
  
  const allContent: ContentItem[] = []
  
  years.forEach(year => {
    const yearDir = path.join(contentDir, year)
    if (fs.statSync(yearDir).isDirectory()) {
      const files = fs.readdirSync(yearDir)
      
      files.forEach(fileName => {
        if (fileName.endsWith('.md')) {
          const fullPath = path.join(yearDir, fileName)
          const fileContents = fs.readFileSync(fullPath, 'utf8')
          const { data, content } = matter(fileContents)
          
          allContent.push({
            id: fileName.replace(/\.md$/, ''),
            content,
            ...data
          } as ContentItem)
        }
      })
    }
  })
  
  return allContent.sort((a, b) => (a.date < b.date ? 1 : -1))
}

export async function renderMarkdown(content: string): Promise<string> {
  const processedContent = await remark()
    .use(remarkGfm)
    .use(html)
    .process(content)
  
  return processedContent.toString()
}
```

4. **Page Components**
```typescript
// app/learnings/page.tsx
import { getAllContent, renderMarkdown } from '@/lib/portfolio'

export default async function LearningsPage() {
  const learnings = getAllContent('learnings')
  
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">Learnings</h1>
      
      <div className="grid gap-6">
        {learnings.map(async (learning) => {
          const contentHtml = await renderMarkdown(learning.content)
          
          return (
            <article key={learning.id} className="bg-white p-6 rounded-lg shadow">
              <h2 className="text-xl font-semibold mb-2">{learning.title}</h2>
              <div className="text-gray-600 mb-4">
                {learning.date} • {learning.category}
              </div>
              <div className="flex flex-wrap gap-2 mb-4">
                {learning.tags.map(tag => (
                  <span key={tag} className="bg-blue-100 text-blue-800 px-2 py-1 rounded text-sm">
                    {tag}
                  </span>
                ))}
              </div>
              <div 
                className="prose prose-sm max-w-none"
                dangerouslySetInnerHTML={{ __html: contentHtml }}
              />
            </article>
          )
        })}
      </div>
    </div>
  )
}
```

5. **Automated Deployment**
```yaml
# .github/workflows/deploy.yml
name: Deploy Portfolio Site

on:
  push:
    branches: [main]
    paths: ['content/**']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: npm install
        
      - name: Build site
        run: npm run build
        
      - name: Deploy to Vercel
        uses: vercel/action@v1
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
```

## Search Integration

### Local Search with ripgrep

Set up powerful local search for your portfolio:

1. **Install ripgrep**
```bash
# macOS
brew install ripgrep

# Ubuntu/Debian
apt install ripgrep

# Windows
choco install ripgrep
```

2. **Search Aliases**
```bash
# Add to .bashrc or .zshrc
alias psearch='rg --type md --heading --line-number --context 2'
alias ptags='rg "tags:" content/ | sed "s/.*tags: *//" | tr -d "[]\"" | tr "," "\n" | sort | uniq -c | sort -nr'
alias precent='find content/ -name "*.md" -mtime -7'
```

3. **Advanced Search Scripts**
```bash
#!/bin/bash
# scripts/portfolio-search.sh

case "$1" in
  "by-tag")
    rg "tags:.*$2" content/ -l
    ;;
  "by-category")
    rg "category: \"$2\"" content/ -l
    ;;
  "recent")
    find content/ -name "*.md" -mtime -"${2:-7}" -exec basename {} .md \;
    ;;
  "content")
    rg "$2" content/ --type md -C 3
    ;;
  *)
    echo "Usage: $0 {by-tag|by-category|recent|content} <term>"
    ;;
esac
```

## Note-Taking App Integration

### Obsidian Integration

Connect your portfolio with Obsidian for graph-based knowledge management:

1. **Vault Setup**
```bash
# Create symbolic links to your content
ln -s /path/to/portfolio/content /path/to/obsidian/vault/Portfolio
```

2. **Obsidian Configuration**
```json
// .obsidian/workspace.json
{
  "main": {
    "id": "main",
    "type": "split",
    "children": [
      {
        "id": "graph-view",
        "type": "leaf",
        "state": {
          "type": "graph",
          "state": {}
        }
      }
    ]
  }
}
```

3. **Custom CSS for Portfolio Content**
```css
/* .obsidian/snippets/portfolio.css */
.portfolio-learning {
  border-left: 4px solid #3b82f6;
  padding-left: 1rem;
}

.portfolio-achievement {
  border-left: 4px solid #10b981;
  padding-left: 1rem;
}

.portfolio-tag {
  background: #f3f4f6;
  padding: 0.25rem 0.5rem;
  border-radius: 0.25rem;
  font-size: 0.875rem;
}
```

### Notion Integration

Sync your portfolio with Notion databases:

1. **Set Up Notion API**
```bash
npm install @notionhq/client
```

2. **Sync Script**
```javascript
// scripts/notion-sync.js
const { Client } = require('@notionhq/client')
const fs = require('fs')
const path = require('path')
const matter = require('gray-matter')

const notion = new Client({ auth: process.env.NOTION_TOKEN })
const databaseId = process.env.NOTION_DATABASE_ID

async function syncToNotion() {
  const learningsDir = path.join(__dirname, '../content/learnings')
  
  // Get all learning files
  const files = getAllMarkdownFiles(learningsDir)
  
  for (const file of files) {
    const content = fs.readFileSync(file, 'utf8')
    const { data, content: markdownContent } = matter(content)
    
    // Create Notion page
    await notion.pages.create({
      parent: { database_id: databaseId },
      properties: {
        Title: { title: [{ text: { content: data.title } }] },
        Tags: { multi_select: data.tags?.map(tag => ({ name: tag })) || [] },
        Date: { date: { start: data.date } },
        Category: { select: { name: data.category } }
      },
      children: [
        {
          object: 'block',
          type: 'paragraph',
          paragraph: {
            rich_text: [{ type: 'text', text: { content: markdownContent.slice(0, 2000) } }]
          }
        }
      ]
    })
  }
}

function getAllMarkdownFiles(dir) {
  const files = []
  const items = fs.readdirSync(dir)
  
  for (const item of items) {
    const fullPath = path.join(dir, item)
    if (fs.statSync(fullPath).isDirectory()) {
      files.push(...getAllMarkdownFiles(fullPath))
    } else if (item.endsWith('.md')) {
      files.push(fullPath)
    }
  }
  
  return files
}

syncToNotion().catch(console.error)
```

## Automation and CI/CD

### GitHub Actions Integration

Automate portfolio maintenance and publishing:

1. **Content Validation**
```yaml
# .github/workflows/validate.yml
name: Validate Portfolio Content

on:
  pull_request:
    paths: ['content/**']

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Check frontmatter
        run: |
          for file in $(find content -name "*.md"); do
            if ! head -10 "$file" | grep -q "title:"; then
              echo "Missing title in $file"
              exit 1
            fi
            if ! head -10 "$file" | grep -q "date:"; then
              echo "Missing date in $file"  
              exit 1
            fi
          done
          
      - name: Check links
        uses: lycheeverse/lychee-action@v1
        with:
          args: --verbose --no-progress 'content/**/*.md'
```

2. **Automated Backups**
```yaml
# .github/workflows/backup.yml
name: Backup Portfolio

on:
  schedule:
    - cron: '0 2 * * 0'  # Weekly on Sunday at 2 AM
  workflow_dispatch:

jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
          
      - name: Create backup
        run: |
          DATE=$(date +%Y%m%d)
          tar -czf "portfolio-backup-$DATE.tar.gz" \
            --exclude='.git' \
            content/ assets/ templates/ docs/
          
      - name: Upload to S3
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
          
      - name: Sync to S3
        run: |
          aws s3 cp portfolio-backup-*.tar.gz s3://your-backup-bucket/portfolio/
```

3. **Social Media Automation**
```yaml
# .github/workflows/social-share.yml
name: Share New Achievements

on:
  push:
    branches: [main]
    paths: ['content/achievements/**']

jobs:
  share:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 2
          
      - name: Get changed files
        id: changed-files
        run: |
          CHANGED=$(git diff --name-only HEAD^ HEAD | grep "content/achievements/")
          echo "files=$CHANGED" >> $GITHUB_OUTPUT
          
      - name: Post to Twitter
        if: steps.changed-files.outputs.files != ''
        uses: ./actions/tweet-achievement
        with:
          twitter-token: ${{ secrets.TWITTER_TOKEN }}
          achievement-file: ${{ steps.changed-files.outputs.files }}
```

### Resume Generation

Automatically generate resumes from your portfolio:

1. **Resume Generator Script**
```python
# scripts/generate-resume.py
import yaml
import json
from jinja2 import Template
from datetime import datetime
import argparse

def load_achievements():
    achievements = []
    # Load and parse achievement files
    # Filter by date range, impact level, etc.
    return achievements

def load_learnings():
    learnings = []
    # Load and parse learning files
    # Group by technology, skill area, etc.
    return learnings

def generate_resume(template_name, output_format):
    data = {
        'achievements': load_achievements(),
        'learnings': load_learnings(),
        'generated_date': datetime.now().strftime('%Y-%m-%d')
    }
    
    with open(f'templates/resume-{template_name}.{output_format}', 'r') as f:
        template = Template(f.read())
    
    rendered = template.render(**data)
    
    with open(f'exports/resume-{template_name}.{output_format}', 'w') as f:
        f.write(rendered)

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--template', default='modern')
    parser.add_argument('--format', default='html')
    
    args = parser.parse_args()
    generate_resume(args.template, args.format)
```

2. **LaTeX Resume Template**
```latex
% templates/resume-modern.tex
\documentclass[letterpaper,11pt]{article}

\title{Professional Resume}
\author{Your Name}

\begin{document}

\section*{Recent Achievements}
{% for achievement in achievements[:5] %}
\subsection*{{ achievement.title }}
\textit{{ achievement.date }} - {{ achievement.impact }} impact

{{ achievement.summary }}

{% if achievement.metrics %}
\begin{itemize}
{% for metric in achievement.metrics %}
\item {{ metric }}
{% endfor %}
\end{itemize}
{% endif %}
{% endfor %}

\section*{Technical Skills}
{% for skill_area in learnings.group_by('category') %}
\textbf{{ skill_area.name }}: 
{% for learning in skill_area.items %}
{{ learning.title }}{% if not loop.last %}, {% endif %}
{% endfor %}
{% endfor %}

\end{document}
```

## Analytics Integration

### Portfolio Analytics Dashboard

Track your learning and growth patterns:

1. **Data Analysis Script**
```python
# scripts/portfolio-analytics.py
import os
import yaml
import matplotlib.pyplot as plt
import pandas as pd
from collections import Counter
from datetime import datetime

def analyze_learning_trends():
    learnings = []
    
    for root, dirs, files in os.walk('content/learnings'):
        for file in files:
            if file.endswith('.md'):
                with open(os.path.join(root, file), 'r') as f:
                    try:
                        content = f.read()
                        if '---' in content:
                            frontmatter = content.split('---')[1]
                            data = yaml.safe_load(frontmatter)
                            learnings.append(data)
                    except:
                        continue
    
    df = pd.DataFrame(learnings)
    
    # Learning frequency over time
    df['date'] = pd.to_datetime(df['date'])
    monthly_counts = df.groupby(df['date'].dt.to_period('M')).size()
    
    plt.figure(figsize=(12, 6))
    monthly_counts.plot(kind='line')
    plt.title('Learning Entries Over Time')
    plt.xlabel('Month')
    plt.ylabel('Number of Entries')
    plt.xtick
```

This guide provides comprehensive integration options for your Portfolio as Code system. Start with the integrations that best match your current workflow and gradually expand as your needs grow.

Remember: the best integration is one that reduces friction and increases the value you get from your portfolio system.

---

*Next: Explore [Team Setup](./team-setup.md) for organizational adoption of Portfolio as Code.*