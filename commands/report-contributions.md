# Report Contributions Command

**Description**: Analyze the uds-core codebase to generate a comprehensive, beautiful HTML report of contributor statistics, commit history, and code changes with interactive visualizations

---

# Contribution Analysis & Reporting Protocol

**Version**: 1.0
**Purpose**: Generate detailed contributor analytics and beautiful HTML reports for the uds-core repository
**Output**: `contribution-report.html` with interactive charts and comprehensive statistics
**Duration**: 15-30 minutes

---

## Overview

This protocol analyzes a Git repository's contribution history to provide insights into:
- 📊 **Contributor Activity**: Who is contributing and how much
- 📈 **Temporal Trends**: How contributions evolve over time
- 🎯 **Impact Metrics**: Lines of code, commits, and file changes
- 👥 **Team Dynamics**: Active contributors and their engagement patterns

**Core Principle**: **Data-driven insights into project health and contributor engagement**

---

## Analysis Process

### Step 1: Repository Preparation (2-5 minutes)

**Ensure Repository is Up-to-Date**:
```bash
# Pull latest changes from remote
git fetch --all
git pull origin main  # or master, depending on default branch

# Verify we have full history
git log --oneline | wc -l
```

**Important**: 
- If the uds-core repository is not cloned locally, clone it first
- Ensure you have access to the complete git history (not a shallow clone)
- Check default branch name (main, master, etc.)

---

### Step 2: Extract Contribution Data (5-10 minutes)

#### 2.1 Gather Raw Git Statistics

**Commands to collect data**:
```bash
# Total commit count
git rev-list --all --count

# Unique contributors
git log --format='%aN' | sort -u | wc -l

# Detailed contributor stats (commits and lines)
git log --all --format='%aN|%aE|%aI' --numstat | \
  awk -F'|' '{
    if (NF == 3) {
      author = $1
      email = $2
      date = $3
    } else if (NF == 3) {
      added += $1
      removed += $2
      files[author]++
    }
  }'

# Per-contributor statistics
git log --all --format='%aN <%aE>' --shortstat --no-merges | \
  awk '/^Author:/ {author=$0} /^ [0-9]/ {print author; print}'

# Monthly contribution data
git log --all --format='%aN|%aI' --numstat | \
  awk -F'|' 'BEGIN {OFS="|"} {
    if (NF == 2) {
      author = $1
      date = substr($2, 1, 7)  # YYYY-MM format
    } else if (NF == 3) {
      added[author][date] += $1
      removed[author][date] += $2
    }
  }'
```

#### 2.2 Identity Deduplication

**Challenge**: Contributors may use multiple email addresses or name variations

**Strategy**:
```python
# Python script for deduplication
identity_map = {
    # Map variations to canonical identity
    "john.doe@company.com": "John Doe",
    "jdoe@company.com": "John Doe",
    "johndoe@gmail.com": "John Doe",
}

# Fuzzy matching for similar names
# - Case-insensitive comparison
# - Handle name order variations (First Last vs Last, First)
# - Common email domain consolidation
```

**Manual Review**: For accuracy, review contributors with similar names and consolidate

---

### Step 3: Data Processing & Analysis (5-10 minutes)

#### 3.1 Calculate Key Metrics

For each contributor:
- **Total Commits**: Count of all commits by this contributor
- **Lines Added**: Sum of all lines added across all commits
- **Lines Removed**: Sum of all lines removed across all commits
- **Net Lines Changed**: Lines Added + Lines Removed
- **Files Modified**: Unique count of files touched
- **First Commit Date**: Earliest commit timestamp
- **Last Commit Date**: Most recent commit timestamp
- **Active Days**: Difference between first and last commit dates
- **Contribution %**: (Net Lines Changed / Total Project Lines Changed) * 100

#### 3.2 Identify Top Contributors

**Rankings**:
1. **Top 10 by Commits**: For timeline visualizations
2. **Top 15 by Commits**: For bar chart visualization
3. **Top 15 by Lines Changed**: For impact visualization
4. **Top 10 for Donut Chart**: For distribution visualization
5. **Top 20 for Detailed Table**: With full metrics

#### 3.3 Generate Time-Series Data

**Monthly Aggregation**:
```python
# For each contributor and each month
monthly_data = {
    "2023-01": {"John Doe": 150, "Jane Smith": 230, ...},
    "2023-02": {"John Doe": 200, "Jane Smith": 180, ...},
    # ...
}

# Cumulative over time
cumulative_data = {
    "2023-01": {"John Doe": 150, "Jane Smith": 230, ...},
    "2023-02": {"John Doe": 350, "Jane Smith": 410, ...},
    # ...
}
```

---

### Step 4: Generate HTML Report (5-8 minutes)

#### 4.1 Report Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>UDS-Core Contribution Report</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
    <style>
        /* Modern, clean styling */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
            line-height: 1.6;
            color: #333;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            padding: 20px;
        }
        
        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            border-radius: 12px;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
            overflow: hidden;
        }
        
        .header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 40px;
            text-align: center;
        }
        
        .header h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
        }
        
        .header .stats-summary {
            display: flex;
            justify-content: center;
            gap: 40px;
            margin-top: 30px;
            flex-wrap: wrap;
        }
        
        .stat-box {
            background: rgba(255, 255, 255, 0.2);
            padding: 20px 30px;
            border-radius: 8px;
            backdrop-filter: blur(10px);
        }
        
        .stat-box .number {
            font-size: 2em;
            font-weight: bold;
            display: block;
        }
        
        .stat-box .label {
            font-size: 0.9em;
            opacity: 0.9;
        }
        
        .section {
            padding: 40px;
            border-bottom: 1px solid #eee;
        }
        
        .section:last-child {
            border-bottom: none;
        }
        
        .section h2 {
            color: #667eea;
            margin-bottom: 20px;
            font-size: 1.8em;
        }
        
        .chart-container {
            position: relative;
            height: 400px;
            margin: 30px 0;
        }
        
        .chart-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
            gap: 30px;
            margin: 30px 0;
        }
        
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
            background: white;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
            border-radius: 8px;
            overflow: hidden;
        }
        
        thead {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        
        th, td {
            padding: 15px;
            text-align: left;
        }
        
        th {
            font-weight: 600;
            text-transform: uppercase;
            font-size: 0.85em;
            letter-spacing: 0.5px;
        }
        
        tbody tr {
            border-bottom: 1px solid #eee;
        }
        
        tbody tr:hover {
            background: #f8f9fa;
        }
        
        tbody tr:last-child {
            border-bottom: none;
        }
        
        .rank {
            display: inline-block;
            width: 30px;
            height: 30px;
            background: #667eea;
            color: white;
            border-radius: 50%;
            text-align: center;
            line-height: 30px;
            font-weight: bold;
            font-size: 0.9em;
        }
        
        .progress-bar {
            height: 8px;
            background: #eee;
            border-radius: 4px;
            overflow: hidden;
            margin-top: 5px;
        }
        
        .progress-fill {
            height: 100%;
            background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
            transition: width 0.3s ease;
        }
        
        .footer {
            padding: 30px 40px;
            background: #f8f9fa;
            text-align: center;
            color: #666;
            font-size: 0.9em;
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Header with Summary Stats -->
        <div class="header">
            <h1>🚀 UDS-Core Contribution Report</h1>
            <p>Comprehensive analysis of contributor activity and impact</p>
            <div class="stats-summary">
                <div class="stat-box">
                    <span class="number" id="total-contributors">0</span>
                    <span class="label">Unique Contributors</span>
                </div>
                <div class="stat-box">
                    <span class="number" id="total-commits">0</span>
                    <span class="label">Total Commits</span>
                </div>
                <div class="stat-box">
                    <span class="number" id="total-lines">0</span>
                    <span class="label">Total Line Changes</span>
                </div>
                <div class="stat-box">
                    <span class="number" id="files-modified">0</span>
                    <span class="label">Files Modified</span>
                </div>
            </div>
        </div>
        
        <!-- Section 1: Timeline Charts -->
        <div class="section">
            <h2>📈 Contribution Timeline</h2>
            <p>Monthly contribution patterns and cumulative impact of top 10 contributors</p>
            
            <div class="chart-container">
                <canvas id="monthlyChart"></canvas>
            </div>
            
            <div class="chart-container">
                <canvas id="cumulativeChart"></canvas>
            </div>
        </div>
        
        <!-- Section 2: Top Contributors Visualizations -->
        <div class="section">
            <h2>🏆 Top Contributors</h2>
            <p>Leading contributors by commits, lines changed, and distribution</p>
            
            <div class="chart-grid">
                <div class="chart-container">
                    <canvas id="commitChart"></canvas>
                </div>
                <div class="chart-container">
                    <canvas id="linesChart"></canvas>
                </div>
            </div>
            
            <div class="chart-container" style="max-width: 600px; margin: 30px auto;">
                <canvas id="donutChart"></canvas>
            </div>
        </div>
        
        <!-- Section 3: Top 20 Contributors Table -->
        <div class="section">
            <h2>📊 Top 20 Contributors (Detailed)</h2>
            <p>Comprehensive metrics with identity deduplication</p>
            
            <table id="top20Table">
                <thead>
                    <tr>
                        <th>Rank</th>
                        <th>Contributor</th>
                        <th>Commits</th>
                        <th>Lines Added</th>
                        <th>Lines Removed</th>
                        <th>Files Changed</th>
                        <th>Contribution %</th>
                    </tr>
                </thead>
                <tbody>
                    <!-- Data populated by JavaScript -->
                </tbody>
            </table>
        </div>
        
        <!-- Section 4: All Contributors Summary -->
        <div class="section">
            <h2>👥 All Contributors Summary</h2>
            <p>Complete contributor roster with activity timeline</p>
            
            <table id="allContributorsTable">
                <thead>
                    <tr>
                        <th>Contributor</th>
                        <th>Total Commits</th>
                        <th>First Commit</th>
                        <th>Last Commit</th>
                        <th>Active Days</th>
                    </tr>
                </thead>
                <tbody>
                    <!-- Data populated by JavaScript -->
                </tbody>
            </table>
        </div>
        
        <div class="footer">
            <p>Generated on <span id="reportDate"></span></p>
            <p>Report generated using the Modernize Agent Framework</p>
        </div>
    </div>
    
    <script>
        // Data will be injected here by the generation script
        const reportData = {
            summary: {
                totalContributors: 0,
                totalCommits: 0,
                totalLines: 0,
                filesModified: 0
            },
            monthly: [],
            cumulative: [],
            topByCommits: [],
            topByLines: [],
            top20: [],
            allContributors: []
        };
        
        // Populate summary statistics
        document.getElementById('total-contributors').textContent = reportData.summary.totalContributors;
        document.getElementById('total-commits').textContent = reportData.summary.totalCommits.toLocaleString();
        document.getElementById('total-lines').textContent = reportData.summary.totalLines.toLocaleString();
        document.getElementById('files-modified').textContent = reportData.summary.filesModified.toLocaleString();
        document.getElementById('reportDate').textContent = new Date().toLocaleDateString();
        
        // Chart.js configuration
        Chart.defaults.font.family = '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif';
        Chart.defaults.color = '#666';
        
        // Color palette for charts
        const colors = [
            '#667eea', '#764ba2', '#f093fb', '#4facfe',
            '#43e97b', '#fa709a', '#fee140', '#30cfd0',
            '#a8edea', '#fed6e3', '#c471f5', '#fa71cd'
        ];
        
        // Monthly Contribution Timeline Chart
        new Chart(document.getElementById('monthlyChart'), {
            type: 'line',
            data: {
                labels: reportData.monthly.labels || [],
                datasets: reportData.monthly.datasets || []
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    title: {
                        display: true,
                        text: 'Monthly Contributions (Top 10 Contributors)',
                        font: { size: 16, weight: 'bold' }
                    },
                    legend: {
                        position: 'bottom',
                        labels: { padding: 15, usePointStyle: true }
                    }
                },
                scales: {
                    y: {
                        beginAtZero: true,
                        title: { display: true, text: 'Lines Changed' }
                    },
                    x: {
                        title: { display: true, text: 'Month' }
                    }
                },
                interaction: {
                    mode: 'index',
                    intersect: false
                }
            }
        });
        
        // Cumulative Contribution Chart
        new Chart(document.getElementById('cumulativeChart'), {
            type: 'line',
            data: {
                labels: reportData.cumulative.labels || [],
                datasets: reportData.cumulative.datasets || []
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    title: {
                        display: true,
                        text: 'Cumulative Contributions Over Time (Top 10)',
                        font: { size: 16, weight: 'bold' }
                    },
                    legend: {
                        position: 'bottom',
                        labels: { padding: 15, usePointStyle: true }
                    }
                },
                scales: {
                    y: {
                        beginAtZero: true,
                        title: { display: true, text: 'Cumulative Lines Changed' }
                    },
                    x: {
                        title: { display: true, text: 'Month' }
                    }
                }
            }
        });
        
        // Top 15 Contributors by Commits
        new Chart(document.getElementById('commitChart'), {
            type: 'bar',
            data: {
                labels: reportData.topByCommits.labels || [],
                datasets: [{
                    label: 'Commits',
                    data: reportData.topByCommits.data || [],
                    backgroundColor: colors[0],
                    borderRadius: 6
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                indexAxis: 'y',
                plugins: {
                    title: {
                        display: true,
                        text: 'Top 15 Contributors by Commits',
                        font: { size: 14, weight: 'bold' }
                    },
                    legend: { display: false }
                },
                scales: {
                    x: {
                        beginAtZero: true,
                        title: { display: true, text: 'Number of Commits' }
                    }
                }
            }
        });
        
        // Top 15 Contributors by Lines Changed
        new Chart(document.getElementById('linesChart'), {
            type: 'bar',
            data: {
                labels: reportData.topByLines.labels || [],
                datasets: [{
                    label: 'Lines Changed',
                    data: reportData.topByLines.data || [],
                    backgroundColor: colors[1],
                    borderRadius: 6
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                indexAxis: 'y',
                plugins: {
                    title: {
                        display: true,
                        text: 'Top 15 Contributors by Lines Changed',
                        font: { size: 14, weight: 'bold' }
                    },
                    legend: { display: false }
                },
                scales: {
                    x: {
                        beginAtZero: true,
                        title: { display: true, text: 'Total Lines Changed' }
                    }
                }
            }
        });
        
        // Contribution Distribution Donut Chart
        new Chart(document.getElementById('donutChart'), {
            type: 'doughnut',
            data: {
                labels: reportData.topByLines.labels?.slice(0, 10) || [],
                datasets: [{
                    data: reportData.topByLines.data?.slice(0, 10) || [],
                    backgroundColor: colors,
                    borderWidth: 2,
                    borderColor: '#fff'
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    title: {
                        display: true,
                        text: 'Contribution Distribution (Top 10)',
                        font: { size: 14, weight: 'bold' }
                    },
                    legend: {
                        position: 'right',
                        labels: { padding: 15, usePointStyle: true }
                    }
                }
            }
        });
        
        // Populate Top 20 Contributors Table
        const top20Body = document.querySelector('#top20Table tbody');
        reportData.top20.forEach((contributor, index) => {
            const row = document.createElement('tr');
            row.innerHTML = `
                <td><span class="rank">${index + 1}</span></td>
                <td><strong>${contributor.name}</strong></td>
                <td>${contributor.commits.toLocaleString()}</td>
                <td>${contributor.linesAdded.toLocaleString()}</td>
                <td>${contributor.linesRemoved.toLocaleString()}</td>
                <td>${contributor.filesChanged.toLocaleString()}</td>
                <td>
                    ${contributor.percentage.toFixed(2)}%
                    <div class="progress-bar">
                        <div class="progress-fill" style="width: ${contributor.percentage}%"></div>
                    </div>
                </td>
            `;
            top20Body.appendChild(row);
        });
        
        // Populate All Contributors Table
        const allContributorsBody = document.querySelector('#allContributorsTable tbody');
        reportData.allContributors.forEach(contributor => {
            const row = document.createElement('tr');
            row.innerHTML = `
                <td><strong>${contributor.name}</strong></td>
                <td>${contributor.commits.toLocaleString()}</td>
                <td>${contributor.firstCommit}</td>
                <td>${contributor.lastCommit}</td>
                <td>${contributor.activeDays} days</td>
            `;
            allContributorsBody.appendChild(row);
        });
    </script>
</body>
</html>
```

#### 4.2 Data Injection Strategy

**Python Script for Report Generation**:
```python
#!/usr/bin/env python3
"""
Generate HTML contribution report for uds-core repository
"""

import subprocess
import json
from datetime import datetime
from collections import defaultdict
import re

def run_git_command(cmd):
    """Execute git command and return output"""
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    return result.stdout

def get_contributor_stats():
    """Extract detailed contributor statistics from git log"""
    # Implementation for gathering all git statistics
    pass

def deduplicate_contributors(contributors):
    """Merge contributors with multiple identities"""
    # Implementation for identity deduplication
    pass

def generate_monthly_data(contributors):
    """Generate monthly contribution time series"""
    # Implementation for monthly aggregation
    pass

def generate_html_report(data, output_file='contribution-report.html'):
    """Generate final HTML report with embedded data"""
    # Read HTML template
    # Inject data as JavaScript
    # Write output file
    pass

if __name__ == '__main__':
    print("Analyzing uds-core repository...")
    stats = get_contributor_stats()
    deduplicated = deduplicate_contributors(stats)
    monthly = generate_monthly_data(deduplicated)
    generate_html_report(deduplicated, 'contribution-report.html')
    print("✅ Report generated: contribution-report.html")
```

---

### Step 5: Review and Validate (2-3 minutes)

**Quality Checks**:
- [ ] Verify all charts render correctly
- [ ] Check that data totals match git statistics
- [ ] Confirm identity deduplication worked properly
- [ ] Validate percentage calculations sum to ~100%
- [ ] Test report in multiple browsers (Chrome, Firefox, Safari)
- [ ] Ensure responsive design works on mobile devices

**Success Criteria**:
- ✅ All 4 sections present and populated
- ✅ Charts are interactive and visually appealing
- ✅ Data accuracy matches git log output
- ✅ Report is professional and presentation-ready
- ✅ No broken functionality or missing data

---

## Implementation Guide

### Required Tools

**Git Analysis**:
```bash
# Ensure git is available
git --version

# Python for data processing (optional, can use other languages)
python3 --version
```

**Alternative Approaches**:
1. **Python Script**: Full automation with data extraction and HTML generation
2. **Bash + JavaScript**: Bash for data extraction, JavaScript for visualization
3. **Git Log + Manual**: Extract data with git commands, manually format

### Example Python Implementation

```python
#!/usr/bin/env python3
"""
UDS-Core Contribution Report Generator

This script analyzes git history and generates a beautiful HTML report
with interactive charts showing contributor statistics and trends.
"""

import subprocess
import json
import re
from datetime import datetime
from collections import defaultdict
from pathlib import Path

class ContributionAnalyzer:
    def __init__(self, repo_path='.'):
        self.repo_path = Path(repo_path)
        self.contributors = defaultdict(lambda: {
            'commits': 0,
            'lines_added': 0,
            'lines_removed': 0,
            'files': set(),
            'first_commit': None,
            'last_commit': None,
            'monthly': defaultdict(int)
        })
        
    def run_git(self, cmd):
        """Execute git command"""
        full_cmd = f"git -C {self.repo_path} {cmd}"
        result = subprocess.run(full_cmd, shell=True, capture_output=True, text=True)
        return result.stdout.strip()
    
    def analyze(self):
        """Main analysis pipeline"""
        print("📊 Analyzing repository...")
        
        # Get all commits with author and date
        log_output = self.run_git(
            'log --all --numstat --format="COMMIT|%aN|%aE|%aI"'
        )
        
        current_author = None
        current_email = None
        current_date = None
        
        for line in log_output.split('\n'):
            if line.startswith('COMMIT|'):
                _, author, email, date = line.split('|')
                current_author = author
                current_email = email
                current_date = datetime.fromisoformat(date.replace('Z', '+00:00'))
                
                # Create identity key
                identity = self.get_identity(author, email)
                
                # Update contributor stats
                contrib = self.contributors[identity]
                contrib['commits'] += 1
                
                if contrib['first_commit'] is None:
                    contrib['first_commit'] = current_date
                contrib['last_commit'] = current_date
                
                # Monthly tracking
                month_key = current_date.strftime('%Y-%m')
                
            elif line and current_author:
                # Parse numstat line: additions deletions filename
                parts = line.split('\t')
                if len(parts) >= 3:
                    added = parts[0]
                    removed = parts[1]
                    filename = parts[2]
                    
                    identity = self.get_identity(current_author, current_email)
                    contrib = self.contributors[identity]
                    
                    if added.isdigit():
                        contrib['lines_added'] += int(added)
                        month_key = current_date.strftime('%Y-%m')
                        contrib['monthly'][month_key] += int(added)
                    
                    if removed.isdigit():
                        contrib['lines_removed'] += int(removed)
                        month_key = current_date.strftime('%Y-%m')
                        contrib['monthly'][month_key] += int(removed)
                    
                    contrib['files'].add(filename)
        
        print(f"✅ Analyzed {len(self.contributors)} unique contributors")
        return self.prepare_report_data()
    
    def get_identity(self, name, email):
        """
        Create canonical identity from name and email.
        This is where deduplication logic goes.
        """
        # Simple approach: use email domain + normalized name
        # More sophisticated: fuzzy matching, manual mapping
        
        # Known aliases (manual mapping)
        alias_map = {
            # Add known aliases here
            # 'old.email@company.com': 'canonical.email@company.com'
        }
        
        if email in alias_map:
            email = alias_map[email]
        
        # Return canonical identity
        return f"{name} <{email}>"
    
    def prepare_report_data(self):
        """Prepare data for HTML report"""
        # Sort contributors by lines changed
        sorted_contribs = sorted(
            self.contributors.items(),
            key=lambda x: x[1]['lines_added'] + x[1]['lines_removed'],
            reverse=True
        )
        
        # Calculate totals
        total_commits = sum(c['commits'] for _, c in sorted_contribs)
        total_lines = sum(
            c['lines_added'] + c['lines_removed'] 
            for _, c in sorted_contribs
        )
        
        # Prepare top 10 for timeline charts
        top10 = sorted_contribs[:10]
        
        # Prepare top 15 for bar charts
        top15_commits = sorted(
            sorted_contribs,
            key=lambda x: x[1]['commits'],
            reverse=True
        )[:15]
        
        top15_lines = sorted_contribs[:15]
        
        # Prepare top 20 for detailed table
        top20 = sorted_contribs[:20]
        
        # Monthly data for charts
        monthly_data = self.build_monthly_datasets(top10)
        cumulative_data = self.build_cumulative_datasets(top10)
        
        return {
            'summary': {
                'totalContributors': len(self.contributors),
                'totalCommits': total_commits,
                'totalLines': total_lines,
                'filesModified': len(set().union(*(c['files'] for _, c in sorted_contribs)))
            },
            'monthly': monthly_data,
            'cumulative': cumulative_data,
            'topByCommits': {
                'labels': [name.split('<')[0].strip() for name, _ in top15_commits],
                'data': [c['commits'] for _, c in top15_commits]
            },
            'topByLines': {
                'labels': [name.split('<')[0].strip() for name, _ in top15_lines],
                'data': [c['lines_added'] + c['lines_removed'] for _, c in top15_lines]
            },
            'top20': [
                {
                    'name': name.split('<')[0].strip(),
                    'commits': c['commits'],
                    'linesAdded': c['lines_added'],
                    'linesRemoved': c['lines_removed'],
                    'filesChanged': len(c['files']),
                    'percentage': ((c['lines_added'] + c['lines_removed']) / total_lines * 100) if total_lines > 0 else 0
                }
                for name, c in top20
            ],
            'allContributors': [
                {
                    'name': name.split('<')[0].strip(),
                    'commits': c['commits'],
                    'firstCommit': c['first_commit'].strftime('%Y-%m-%d') if c['first_commit'] else 'N/A',
                    'lastCommit': c['last_commit'].strftime('%Y-%m-%d') if c['last_commit'] else 'N/A',
                    'activeDays': (c['last_commit'] - c['first_commit']).days if c['first_commit'] and c['last_commit'] else 0
                }
                for name, c in sorted_contribs
            ]
        }
    
    def build_monthly_datasets(self, top_contributors):
        """Build monthly contribution datasets for Chart.js"""
        # Get all months
        all_months = set()
        for _, contrib in top_contributors:
            all_months.update(contrib['monthly'].keys())
        
        sorted_months = sorted(all_months)
        
        # Build datasets
        datasets = []
        colors = [
            '#667eea', '#764ba2', '#f093fb', '#4facfe',
            '#43e97b', '#fa709a', '#fee140', '#30cfd0',
            '#a8edea', '#fed6e3'
        ]
        
        for idx, (name, contrib) in enumerate(top_contributors):
            dataset = {
                'label': name.split('<')[0].strip(),
                'data': [contrib['monthly'].get(month, 0) for month in sorted_months],
                'borderColor': colors[idx % len(colors)],
                'backgroundColor': colors[idx % len(colors)] + '33',
                'tension': 0.4
            }
            datasets.append(dataset)
        
        return {
            'labels': sorted_months,
            'datasets': datasets
        }
    
    def build_cumulative_datasets(self, top_contributors):
        """Build cumulative contribution datasets"""
        # Get all months
        all_months = set()
        for _, contrib in top_contributors:
            all_months.update(contrib['monthly'].keys())
        
        sorted_months = sorted(all_months)
        
        # Build cumulative datasets
        datasets = []
        colors = [
            '#667eea', '#764ba2', '#f093fb', '#4facfe',
            '#43e97b', '#fa709a', '#fee140', '#30cfd0',
            '#a8edea', '#fed6e3'
        ]
        
        for idx, (name, contrib) in enumerate(top_contributors):
            cumulative = 0
            cumulative_data = []
            for month in sorted_months:
                cumulative += contrib['monthly'].get(month, 0)
                cumulative_data.append(cumulative)
            
            dataset = {
                'label': name.split('<')[0].strip(),
                'data': cumulative_data,
                'borderColor': colors[idx % len(colors)],
                'backgroundColor': colors[idx % len(colors)] + '33',
                'tension': 0.4,
                'fill': False
            }
            datasets.append(dataset)
        
        return {
            'labels': sorted_months,
            'datasets': datasets
        }

def generate_html_report(data, output_path='contribution-report.html'):
    """Generate HTML report with embedded data"""
    
    # Read the HTML template (embedded above)
    html_template = """
    [FULL HTML TEMPLATE GOES HERE]
    """
    
    # Inject data by replacing placeholder
    html_with_data = html_template.replace(
        'const reportData = {',
        f'const reportData = {json.dumps(data, indent=2)};'
    ).replace('};', '', 1)  # Remove the closing brace we just added
    
    # Write output file
    with open(output_path, 'w') as f:
        f.write(html_with_data)
    
    print(f"✅ Report generated: {output_path}")

if __name__ == '__main__':
    print("🚀 UDS-Core Contribution Report Generator")
    print("=" * 50)
    
    # Analyze repository
    analyzer = ContributionAnalyzer('.')
    data = analyzer.analyze()
    
    # Generate HTML report
    generate_html_report(data)
    
    print("\n✅ Report generation complete!")
    print("📄 Open contribution-report.html in your browser to view")
```

---

## Usage Examples

### Example 1: Generate Report for UDS-Core

```bash
# Command
/report-contributions

# Claude's Response
I'll analyze the uds-core repository and generate a contribution report...

Step 1: Checking repository status...
✅ Repository is up to date
✅ Full git history available (2,847 commits)

Step 2: Extracting contribution data...
📊 Analyzing 2,847 commits
👥 Found 47 unique contributors
📝 Tracked 156,234 line changes
📁 Modified 1,234 files

Step 3: Processing data...
🔄 Deduplicating contributor identities
📈 Building monthly timelines
🎯 Calculating contribution percentages

Step 4: Generating HTML report...
✅ Created contribution-report.html

Summary:
- Unique Contributors: 47
- Total Commits: 2,847
- Total Line Changes: 156,234
- Files Modified: 1,234

Top 3 Contributors:
1. John Doe - 543 commits (19.1%)
2. Jane Smith - 412 commits (14.5%)
3. Bob Johnson - 387 commits (13.6%)

📄 Open contribution-report.html in your browser to view the full interactive report
```

---

## Best Practices

### Data Quality

1. **Ensure Complete History**: Use full clones, not shallow clones
2. **Update Regularly**: Pull latest changes before analysis
3. **Manual Review**: Verify identity deduplication results
4. **Document Assumptions**: Note any manual adjustments made

### Report Generation

1. **Test Locally**: Open HTML file before sharing
2. **Browser Compatibility**: Test in Chrome, Firefox, Safari
3. **Mobile Responsive**: Verify charts work on mobile devices
4. **Performance**: Optimize for large datasets (>1000 contributors)

### Identity Deduplication

1. **Email Domains**: Group by company email domain
2. **Name Variations**: Handle "John Doe" vs "Doe, John"
3. **Case Sensitivity**: Normalize to lowercase
4. **Manual Mapping**: Maintain alias file for known duplicates

---

## Troubleshooting

### Common Issues

**Issue**: "Not a git repository"
- **Solution**: Ensure you're in the repository directory or clone uds-core first

**Issue**: Empty charts or missing data
- **Solution**: Check git log output, verify commits exist in specified branch

**Issue**: Contributors with multiple entries
- **Solution**: Update identity deduplication logic with known aliases

**Issue**: Report not rendering in browser
- **Solution**: Check browser console for JavaScript errors, verify Chart.js CDN is accessible

**Issue**: Slow performance with large repositories
- **Solution**: Add pagination to tables, limit time range, or sample data

---

## Advanced Features (Optional)

### Extended Analysis

1. **Code Quality Metrics**: Integrate complexity analysis
2. **Commit Message Analysis**: Categorize commits by type (feature, fix, docs)
3. **Review Participation**: Track PR reviews and comments
4. **Team Velocity**: Calculate sprint-based velocity metrics
5. **Expertise Mapping**: Identify experts by file/directory contributions

### Export Options

1. **CSV Export**: Generate spreadsheet-friendly format
2. **PDF Report**: Convert HTML to PDF for sharing
3. **JSON API**: Export raw data for integration
4. **Dashboard Integration**: Connect to existing dashboards

---

**Command Owner**: Documentation Agent
**Protocol Version**: 1.0
**Last Updated**: 2025-11-17
**Dependencies**: Git, Python 3.7+ (or bash + JavaScript)

**Remember**: **This report provides insights for recognition, not surveillance. Use it to celebrate contributions and identify areas for team growth.** ✅
