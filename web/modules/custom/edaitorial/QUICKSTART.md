# 🚀 edAItorial - Quick Start Guide

## ✅ Module Overview

**edAItorial** (Editorial + AI) is your AI-powered editorial assistant that helps you create better content with real-time SEO and accessibility insights.

## 🎯 What Does It Do?

- 📊 **Dashboard Analytics** - Visual insights into your site's SEO and accessibility health
- 🔍 **Content Analysis** - Pre-publish checks for every piece of content
- ♿ **WCAG Compliance** - Track accessibility across Levels A and AA
- 🤖 **AI Integration Ready** - Built for amazee.io AI integration

## 🚀 Quick Installation

### 1. Enable the Module

```bash
ddev drush en edaitorial -y
ddev drush cr
```

### 2. Configure Permissions

Navigate to: `/admin/people/permissions`

Assign these permissions:
- **View edAItorial Dashboard** → Editors, Administrators
- **Administer edAItorial** → Administrators only

### 3. Access the Dashboard

Navigate to: **Configuration → Content authoring → edAItorial**

Or directly: `/admin/config/content/edaitorial`

## 📊 Dashboard Features

### Main Dashboard
- **Overall Health Score** (0-100) - Combines SEO + Accessibility
- **Pages Crawled** - Total published content analyzed
- **SEO Issues** - Number of SEO problems detected
- **A11y Issues** - Accessibility issues found
- **Avg. Load Time** - Performance metric

### SEO Health Checklist
✅ Meta Titles  
✅ Meta Descriptions  
✅ Canonical URLs  
✅ XML Sitemap  
✅ Robots.txt  
✅ Structured Data  
✅ Open Graph Tags  
✅ Mobile Friendly  

### WCAG Compliance
- **Level A** (Minimum) - 4 principles
- **Level AA** (Recommended) - 4 principles
- Real-time progress tracking

## 🎨 Using edAItorial

### For Editors

1. **Create/Edit Content**
2. **Find the edAItorial section** in the sidebar
3. **Click "Analyze Content"**
4. **Review suggestions**:
   - SEO optimization tips
   - Accessibility issues
   - Readability improvements
5. **Make improvements**
6. **Publish with confidence!**

### For Administrators

1. **Access Dashboard**: `/admin/config/content/edaitorial`
2. **Run Site Audit**: Click "Run Audit" button
3. **Review Metrics**:
   - Overall health score
   - Issue breakdown
   - Recent activity
4. **Configure Settings**: `/admin/config/content/edaitorial/settings`
   - Enable/disable pre-publish checks
   - Set title length limits
   - Choose WCAG target level

## 🤖 AI Integration (amazee.io)

edAItorial is ready for AI integration:

```php
// In src/Service/ContentAnalyzer.php
protected function getAiSuggestions(NodeInterface $node) {
  $ai_provider = \Drupal::service('ai.provider.amazeeio');
  
  $prompt = "Analyze this content:\n\n";
  $prompt .= "Title: {$node->getTitle()}\n";
  $prompt .= "Content: {$node->get('body')->value}\n";
  
  return $ai_provider->chat($prompt)->getSuggestions();
}
```

## 📁 File Structure

```
edaitorial/
├── edaitorial.info.yml         # Module info
├── edaitorial.routing.yml      # 5 routes
├── edaitorial.permissions.yml  # 2 permissions
├── edaitorial.services.yml     # 4 services
├── src/
│   ├── Controller/             # Dashboard controller
│   ├── Form/                   # Settings form
│   └── Service/                # 4 analyzer services
├── templates/                  # 4 Twig templates
├── css/                        # Dashboard styles
└── js/                         # Dashboard interactions
```

## 🔧 Configuration Options

### Settings Page
Navigate to: `/admin/config/content/edaitorial/settings`

**Analysis Settings:**
- ☑ Enable pre-publish content check
- ☑ Enable AI-powered suggestions

**SEO Settings:**
- Min title length: 30 characters
- Max title length: 60 characters

**Accessibility Settings:**
- Target WCAG level: A, AA, or AAA

## 🎯 Navigation

| Page | Path |
|------|------|
| Dashboard | `/admin/config/content/edaitorial` |
| SEO Overview | `/admin/config/content/edaitorial/seo` |
| Accessibility | `/admin/config/content/edaitorial/accessibility` |
| Content Audit | `/admin/config/content/edaitorial/content-audit` |
| Settings | `/admin/config/content/edaitorial/settings` |

## 💡 Tips

### For Best Results

1. **Run audits regularly** - Stay on top of issues
2. **Analyze before publishing** - Catch problems early
3. **Set realistic targets** - WCAG Level AA is recommended
4. **Train your team** - Show editors how to use analysis tools
5. **Monitor trends** - Track improvements over time

### Performance

- Metrics are calculated on-demand
- Consider caching for large sites
- Run full audits during off-peak hours

## 🐛 Troubleshooting

### Module not showing up?
```bash
ddev drush en edaitorial -y
ddev drush cr
```

### Dashboard not accessible?
Check permissions at `/admin/people/permissions`

### Styles not loading?
```bash
ddev drush cr
# Check that library is defined in edaitorial.libraries.yml
```

### PHP errors?
```bash
ddev drush watchdog:show --severity=Error
```

## 🎉 Quick Commands

```bash
# Enable module
ddev drush en edaitorial -y

# Clear cache
ddev drush cr

# Open dashboard
ddev launch /admin/config/content/edaitorial

# Check module status
ddev drush pm:list | grep edaitorial

# Uninstall (if needed)
ddev drush pm:uninstall edaitorial -y
```

## 📚 Learn More

- Full documentation: `README.md`
- Module location: `web/modules/custom/edaitorial/`
- Support: Contact the development team

---

**Built for the DrupalAI Hackathon 2026** 🚀

*Making editorial work smarter with AI* ✨
