# **Modernization Plan for the Luke Cyr Foundation WordPress Theme**

This plan outlines a step-by-step modernization strategy to rebuild the Luke Cyr Foundation’s WordPress theme with a **block-first architecture (Full Site Editing)**, improved performance, accessibility, and maintainability. Each phase is detailed with tactical steps and justifications, covering everything from architectural changes to CI/CD workflow.

## **Phase 1: Architectural Overhaul – From PHP Templates to Full Site Editing (Block Theme)**

**Goal:** Convert the legacy PHP-based theme into a **Full Site Editing (FSE)** block theme for greater flexibility and future-proofing.

* **Scaffold a New Block Theme:** Create a new theme folder (e.g. `lcf-block-theme`) with the required files: a basic `style.css` (for theme metadata) and a `templates/index.html` as the starting point. Include a `parts/` directory for global template parts (header, footer) and `templates/` for page templates.

* **Rebuild Templates in HTML:** Replace PHP templates (`header.php`, `footer.php`, `page.php`, etc.) with block-based HTML templates. For example, convert the header and footer into `parts/header.html` and `parts/footer.html` containing block markup (Site Logo block, Navigation block, etc.), and create `templates/page.html`, `templates/single.html`, `templates/front-page.html` as needed to mirror key layouts. This decouples content structure from code and lets editors rearrange layout via the Site Editor.

* **Leverage Core Blocks & Patterns:** Use WordPress core blocks instead of custom PHP where possible. For navigation menus, use the **Navigation Block** (replacing any custom menu PHP). For dynamic content like post listings (e.g. “Success Stories”), use a Query Loop block in a template or pattern instead of a custom query code. Build **custom block patterns** for frequently used sections (hero banners, call-to-action sections) so editors can insert them easily. This ensures layout consistency and reusability without hardcoding in templates.

* **Incorporate Component-Based Structure:** Refactor repeating elements into reusable components. For instance, if the site has a “card” design for profiles or stories, create a block pattern or custom block for it, rather than duplicating HTML/CSS in multiple places. The new theme should treat elements like buttons, cards, and banners as single components that can be updated globally. This **design system** approach means the "Donate" button and "Subscribe" button will be the same component, improving consistency and maintainability.

* **Justification:** Moving to an FSE block theme dramatically improves editorial control and agility. Content managers can rearrange or update sections without developer intervention, addressing the legacy theme’s rigidity. It also sheds legacy bloat – block themes are generally lighter and faster by relying on native WordPress block functionality instead of large custom PHP codebases.

## **Phase 2: Modernize Styling – From SCSS/Bootstrap to a Lean CSS Strategy**

**Goal:** Simplify and optimize the CSS framework by either using a utility-first approach or lean SCSS, dropping unnecessary bloat for better performance and maintainability.

* **Audit Current Styles:** Review the existing SCSS/Bootstrap usage to identify which components and grid styles are actually used. This allows a decision on keeping SCSS or migrating to utility-first CSS. If Bootstrap 5 is currently included, consider removing unused components from the build or dropping Bootstrap entirely if its features are not critical.

* **Utility-First CSS Option:** Consider adopting a utility-first CSS framework (e.g. **Tailwind CSS**). Utility classes can greatly reduce CSS file size by only including what’s used, and speed up development with predefined classes for spacing, typography, colors, etc. The trade-off is more classes in HTML and a learning curve, but it yields a highly optimized CSS payload and consistent styling. If utility-first is chosen, set up a build process to purge unused classes in production for minimal CSS.

* **Lean SCSS Option:** If sticking with SCSS, refactor the styles into modular partials (e.g. `_buttons.scss`, `_header.scss`) rather than one large file. Include only necessary Bootstrap parts (using Sass imports to include just the grid, reboot, or specific components needed). This keeps the CSS lean and focused. Define common variables or use CSS Custom Properties for brand colors, etc., ideally sourced from the theme’s design tokens (see theme.json in Phase 3).

* **Native CSS & Design Tokens:** Leverage modern CSS features for maintainability. Use **CSS custom properties** for colors, fonts, and spacing that align with theme design tokens defined in `theme.json`. This makes it easy to tweak styles centrally. For example, define `--color-primary` in theme.json and use it in the CSS; updating the color in one place updates it everywhere.

* **Eliminate jQuery-dependent CSS/JS:** If any CSS relies on jQuery (e.g. for collapse menus or modals from Bootstrap), replace those interactions with pure CSS or vanilla JS alternatives. For example, use CSS `:target` or checkbox hack for simple show/hide, or lightweight JS for toggles, instead of a full jQuery \+ Bootstrap JS bundle.

* **Justification:** Removing or minimizing heavy frameworks like Bootstrap reduces file size and improves load times. A **lean CSS architecture** (whether utility-first or modular SCSS) means less unused CSS ships to the browser, boosting performance on mobile. It also aids maintainability – future developers can easily find and update a small set of style definitions rather than overriding a giant stylesheet. By using theme.json and CSS variables, we ensure consistent theming without duplicating magic numbers or colors in multiple places.

## **Phase 3: Performance Optimization and Core Web Vitals**

**Goal:** Achieve fast load times and good Core Web Vitals (LCP, FID/INP, CLS) through efficient asset loading, image optimization, and script management.

* **Optimize Images:** Convert heavy images (especially large PNGs or hero images) to modern formats like **WebP or AVIF**. Implement responsive images with `srcset` and `sizes` so each device gets an appropriately sized image, preventing huge downloads on mobile. For example, the hero/banner image should be served as a compressed WebP and have multiple size variants for different screen widths. This directly improves the LCP metric by cutting image load time dramatically.

* **Minimize Render-Blocking Resources:** Audit CSS and JS includes. **Inline critical CSS** for above-the-fold content (e.g. critical styles for header, navigation, hero section) and load the rest of the CSS asynchronously. This ensures the user sees a styled header immediately, with no flash of unstyled content, improving perceived performance. Likewise, move all non-critical scripts out of the `<head>`; use the `defer` attribute or load in footer for anything that isn’t needed for initial paint. For instance, contact form validation scripts or third-party widgets should be deferred so they don't block page rendering.

* **Remove jQuery and Legacy Scripts:** If feasible, **remove jQuery** from the frontend to eliminate an extra 80–100KB parse/block. Replace any jQuery-dependent code with vanilla JavaScript. Modern browsers and WordPress APIs allow DOM manipulation and AJAX without jQuery, and many libraries now have pure JS alternatives. If a plugin still requires jQuery, ensure it's loaded with `defer` so it doesn’t stall the page. *Justification:* Eliminating jQuery can significantly cut down main-thread work and improve First Input Delay (or INP) by not loading an unnecessary library on every page.

* **Conditional & Lazy Asset Loading:** Implement a strategy where assets are loaded only on the pages that need them (**minimal, conditional loading**). For example, if the “Road to Resilience” page has a video gallery, only enqueue the video player script on that page, not site-wide. Leverage WordPress’s conditional tags or block asset enqueuing: if a custom block is present on a page, its script/styles get loaded; otherwise not. This might involve splitting the theme’s JS/CSS bundle by functionality. **Example:** Only load the donation form JS on the Donate page, Google Maps API only on pages that show a map, etc. The previous theme likely loaded all JS/CSS on every page, which we will correct by auditing `functions.php` enqueues. This *conditional loading* reduces unnecessary bytes for most pageviews, boosting performance.

* **Third-Party Embed Optimization:** Apply a “**facade**” or on-demand loading pattern for heavy third-party content. For instance, on the “Documentary” page with an embedded YouTube video, initially show a static preview image with a play button. Only load the actual YouTube embed/player script after the user clicks play. This defers \~500KB of YouTube iframe script until needed, vastly reducing initial load payload. Similarly, defer loading of social media widgets or large analytics scripts until after the main content is interactive.

* **Core Web Vitals Monitoring:** After making these changes, continuously monitor performance (use Lighthouse or WebPageTest on staging). Aim for LCP under \~2.5s on mobile, CLS \< 0.1 (ensure no layout shifts), and a good INP by keeping main-thread JS light. Use browser caching or a plugin for page caching (if allowed, though minimal plugins, but a must-have caching solution or server-side cache can help achieve consistent performance for returning visitors).

* **Justification:** For the Foundation’s audience, performance is critical – a slow site can literally be a barrier to someone seeking help. The above tactics (optimized images, no render-blocking JS/CSS, and only loading what’s needed) directly improve **Core Web Vitals** and ensure even users on slow networks or older devices can access the site quickly. Faster load times also improve SEO and overall user engagement.

## **Phase 4: Accessibility Upgrades (WCAG 2.2 Compliance)**

**Goal:** Achieve at least WCAG 2.2 AA accessibility compliance across the site, ensuring users of all abilities can navigate and use the site’s features.

* **Semantic Structure & Landmarks:** Use proper HTML5 semantic elements (header, nav, main, footer, etc.) in templates so assistive technologies can navigate regions easily. For critical info like the crisis hotline banner, use `role="alert"` or an ARIA live region so screen readers announce it prominently. Ensure that banner’s phone number is a clickable tel: link for direct action on mobile.

* **Form Labeling and Inputs:** Every form field on “Donate” or “Contact” pages must have a corresponding `<label>` (or an aria-label if a visual label isn’t possible). The current theme may have used placeholder text as labels, which is insufficient. We will add explicit labels and descriptions for inputs (e.g. for accessibility, a hidden “Required” note if needed). This fixes a WCAG 2.2 issue and helps users with cognitive or memory impairments not lose context when typing. Also, implement meaningful error messages and consider features like **redundant entry prevention** – for example, if a user already entered their email in a subscribe form, pre-fill it on donate page to reduce re-typing.

* **Keyboard Navigation & Focus:** Ensure the entire site is operable via keyboard alone. This means **visible focus indicators** on all interactive elements. We will **restore focus styles** that might have been removed; for instance, add a high-contrast focus ring (e.g. a thick outline in the Foundation’s gold/blue) around links, buttons, form fields when focused. The default browser focus outline should not be disabled unless replaced with an equivalent visible style. Test by tabbing through menus, links, and forms to confirm the focus order is logical and all controls can be activated by keyboard.

* **Alt Text and Media:** Audit all images and media for alternative text. All meaningful images need descriptive `alt` attributes. For example, instead of `alt="logo.png"`, use something like `alt="Luke Cyr Foundation logo"` or descriptive text for content images (e.g. `alt="Veteran receiving support at a Foundation event"`). Decorative images should have empty alt (`alt=""`) so they’re ignored by assistive tech. Also ensure videos have captions or transcripts available. These measures make content perceivable for users with visual/hearing disabilities.

* **Skip Navigation Links:** Implement a “**Skip to Main Content**” link as the first focusable element on the page. This allows keyboard and screen reader users to bypass repetitive navigation menus quickly. It can be visually hidden until focused (for sighted keyboard users).

* **Color Contrast and Theming:** Use the theme’s color palette in a way that maintains **high contrast**. Verify all text/background color combinations meet WCAG 2.2 contrast ratios (at least 4.5:1 for normal text). Adjust theme colors if needed or add a darker overlay under white text on hero images, etc. Aim for some areas to reach AAA (7:1) where possible for body text to account for older users with low vision. We will test pages with contrast checker tools and adjust CSS or theme.json values to ensure accessible color contrast.

* **Testing & Validation:** Utilize tools like WAVE, AXE, or Lighthouse accessibility tests on the staging site. Also perform manual testing with screen reader software (NVDA/VoiceOver) to catch issues in reading order or missing context. Iterate on any issues found.

* **Justification:** Many of the Foundation’s users may have disabilities (e.g. vision impairments, PTSD-related cognitive difficulties), so an accessible site is mission-critical, not just nice-to-have. Conforming to WCAG 2.2 AA ensures the site can be used by the widest audience. These improvements (proper labels, focus states, alt text, etc.) also enhance overall UX and often SEO. An accessible site is more robust and effective in delivering the Foundation’s content to everyone.

## **Phase 5: Plugin Audit and “Wall of Honor” Integration**

**Goal:** Minimize reliance on plugins while retaining needed functionality, and decide how to handle the **“Wall of Honor”** feature (a critical part of the site content).

* **Audit Existing Plugins:** Make a list of all current plugins and their purposes. For each, determine if it’s truly necessary or if the feature can be achieved with core WordPress or custom code. Remove any plugins that are redundant or no longer used to slim down the site (each plugin can introduce performance overhead or security risk).

* **Wall of Honor Plugin – Keep or Rebuild:** The *Wall of Honor* is a key page (currently powered by a plugin). We need to decide its fate:

  * **Option A: Keep & Update** – If this plugin is custom-built for listing honorees (e.g. a custom post type or shortcode that outputs a list of names/donors), it may be simpler to keep it. Ensure the plugin is **updated for PHP 8+ compatibility and block theme support** (no hardcoded HTML that conflicts with new markup). We might need to adjust its template or use its data within a block. For example, if it provides a shortcode, create a custom block that wraps that output for easier placement in the new theme.

  * **Option B: Migrate/Rebuild** – If the plugin is outdated or limiting, rebuild the Wall of Honor functionality using native WordPress features. One approach: define a **custom post type** for Honorees (if not already), and use a block (Query Loop or a custom block) to display them. This way, the listing can be managed within the block editor (e.g. each honoree as a post with metadata). We can then drop the old plugin. Rebuilding might involve writing a simple custom plugin or including CPT registration in `functions.php`, plus creating a template (e.g. `archive-honoree.html` for listing, `single-honoree.html` for details) in the block theme. This yields full control over markup and styling in the new system.

  * **Option C: Static Content** – If the Wall of Honor content is relatively static and doesn’t require dynamic updates or user submissions, it might even be turned into a static page in the block editor with a nice design (using group blocks, columns, etc., to list honorees). This eliminates the plugin entirely. However, if names are frequently added/removed, a dynamic solution (Option A or B) is better for ease of updates.

* **Decision Criteria:** Evaluate how complex the Wall of Honor data is and who updates it. If it’s essentially a list of individuals and bios, the CPT \+ block approach (Option B) could be ideal for long-term maintainability (no third-party dependency, all under our control). If time is short and the existing plugin works fine, keeping it (Option A) is acceptable but ensure it adheres to our new standards (no jQuery, sanitized output, matches theme styles). Document the decision in the code repository for clarity.

* **Other Critical Pages:** Ensure that pages like **Donate, Success Stories, Road to Resilience** are handled optimally:

  * *Donate:* If using a donation plugin or embed (e.g. PayPal or GiveWP), try to use minimal embed code. If it’s a custom form, ensure it is secure and maybe integrate with an API rather than heavy plugins. Possibly this page can be largely static content with a form block or embed code provided by a payment processor, reducing the need for a plugin.

  * *Success Stories:* Likely a listing of stories (perhaps blog posts or a category). Implement this with core blocks (e.g., a Query Loop for a category “Success Story”) or a page template. Remove any old page-builder or shortcode usage and replace with clean block templates. Ensure media in success stories is handled with the performance and accessibility measures above (e.g. images optimized, transcripts for any video).

  * *Road to Resilience:* If this is a special campaign page, it might have had custom page templates. Recreate it in the block editor using group blocks, banners, etc., or a custom page template in the block theme for that page specifically. This avoids needing a plugin for just layout; the block editor can handle it with custom design.

* **Minimal Plugins Principle:** After modernization, the site should ideally rely only on a handful of well-supported plugins: e.g., a **security plugin** (if used for firewall or 2FA), maybe a **form plugin** (if the built-in block forms are insufficient), and possibly an **SEO plugin**. Everything else (navigation, sliders, galleries, etc.) can often be handled with core blocks or lightweight custom code. Fewer plugins mean fewer updates and less risk of conflicts, aligning with our performance and security goals.

* **Justification:** Each plugin removed is one less potential point of failure or vector for slowdown. By handling the Wall of Honor in theme/core, we gain full control over its performance and markup. If the Foundation’s deployment is internal, we’re not constrained by off-the-shelf solutions – we can tailor functionality to exactly what’s needed. This phase ensures we **keep necessary features** (honoring donors, etc.) but **trim away bloat**, yielding a faster, more secure site.

## **Phase 6: Component-Based Patterns & Semantic Markup**

**Goal:** Implement a design system approach using WordPress block patterns and ensure all markup is clean and semantic for longevity.

* **Design System & Patterns:** Identify repeating design elements (calls to action, profile cards, event highlights, etc.) and implement them as **block patterns** or reusable blocks. For example, create a pattern for a “Success Story Card” that includes an image, title, excerpt, and a read-more link with a consistent style. Editors can insert this pattern whenever adding a new story, ensuring the layout is uniform. This component approach was not possible in the old theme without custom code, but now the theme can define patterns in code (in a `patterns/` folder) that appear in the editor. It reduces error and speeds up content creation.

* **Reusable Blocks vs Template Parts:** Use **reusable blocks** for content that might repeat across pages (e.g., a donation banner used in multiple places). For global areas like headers or footers, we already have template parts in Phase 1\. For section patterns that might slightly vary, block patterns (which insert copyable content) are ideal.

* **Enforce Semantic HTML:** Ensure all custom block markup or pattern HTML uses semantic tags. For instance, use lists (`<ul>`, `<li>`) for listing honorees or resources, use `<section>` with appropriate ARIA labels for distinct sections of a page, use `<article>` for each Success Story, etc. This not only improves accessibility but also SEO (search engines understand the content structure better).

* **No Presentational Markup:** Strip out any legacy presentational code (e.g. `<br>` for spacing, tables for layout). The new theme should rely on CSS for layout and spacing (via margins, flex, grid, etc., which can be configured via block controls or style classes). This makes the HTML cleaner and the design easier to adjust globally.

* **Microdata/Schema (Optional):** As a forward-looking enhancement, incorporate structured data where appropriate. For example, mark the organization info (address, etc.) with schema.org **NonProfitOrganization** metadata, or mark “Success Stories” as BlogPosting schema. This wasn’t requested explicitly, but adding it during the markup revamp could improve SEO without affecting front-end (since it’s just `<script type="application/ld+json">` data or itemprop attributes). It’s a “nice-to-have” that aligns with modernization.

* **Justification:** A component-based design with semantic HTML ensures the site is **maintainable and scalable**. If a style needs to change (say, how a profile card looks), updating the pattern or CSS in one place will update all instances, avoiding divergent styling. Semantic markup is future-proof: it will work with screen readers, search engine crawlers, and any new user agents. This phase complements Phase 1 (architecture) by refining *how* we build the blocks and templates in a consistent, standard way. It lays a solid foundation for any future redesigns or feature additions, as everything follows a predictable pattern library.

## **Phase 7: CI/CD and Sustainable Deployment Workflow**

**Goal:** Establish a robust development workflow with version control, code review, and continuous integration/continuous deployment (CI/CD) to maintain code quality and site stability.

* **Version Control with Branching:** Maintain the theme (and any custom plugin code like “Wall of Honor”) in a Git repository. Adopt a branching strategy such as **Git Flow** or GitHub flow for organized development. For example:

  * Use a permanent **main** branch for production-ready code (the live site runs this).

  * Have a **develop** branch for accumulating the next release (if multiple features are being worked on).

  * Create feature branches (e.g. `feature/wall-of-honor-refactor` or `feature/accessibility-fixes`) for each task or bugfix, which then get merged into develop/main via pull requests.

* **Pull Request Reviews:** Enforce that all changes go through a Pull Request (PR) and at least one other developer reviews the code before merging. This practice (the “four eyes principle”) catches mistakes and enforces coding standards. Even in a small team, PRs are useful to document changes and have a history of discussions for why something was done.

* **Coding Standards:** Use automated tools to maintain WordPress Coding Standards (WPCS). For PHP, run a linter or PHPCS in the CI pipeline to flag any deviations (especially important as we refactor for PHP 8 compatibility and proper escaping/sanitization). For CSS/JS, linters or formatters (ESLint, Prettier, Stylelint) can be integrated to keep code style consistent.

* **CI Pipeline:** Implement a CI pipeline (e.g. using **GitHub Actions** or another CI service) that runs on each push or PR. The pipeline can run tasks like:

  * **Build:** If using a build system (for SCSS or Tailwind, etc.), have CI compile the assets to ensure no build errors.

  * **Test:** Run any automated tests. For a theme, this might include visual regression tests or basic PHPUnit tests if any PHP functions exist. At minimum, run a syntax check and the linters mentioned.

  * **Deploy:** Set up automatic deployment to a **staging environment** when code is merged into the develop branch (or a specific branch). For example, merging to `staging` branch triggers a deployment to a staging site (on an internal server). After verification, merging to `main` could auto-deploy to production (with appropriate safeguards). This removes human error from FTPing files and ensures deployments are consistent.

* **Staging Environment & Testing:** Maintain an up-to-date **staging site** that mirrors production. Before any major update (theme release, plugin update, WP core update), test on staging first. The workflow would be: Backup live \-\> deploy changes to staging \-\> test everything (pages load, forms work, no console errors, etc.) \-\> then deploy to live. If an issue is found, fix it in code and iterate on staging until resolved, then go live. This prevents the live site from ever showing fatal errors or broken functionality to users.

* **Branch Discipline:** Encourage developers (even if it’s a small team or a single developer wearing multiple hats) to avoid quick fixes on the live server. All changes should go through Git and CI. This discipline pays off by creating a **trackable history** and the ability to rollback if a deployment causes an issue (since you can revert a commit). It also means any hotfix done in production is back-ported into Git so nothing is lost.

* **Justification:** A sustainable workflow is essential for the *long-term success* of the theme. The site is internal now, but treating it with the rigor of a software project ensures that as it grows (or if new developers join), there is confidence in making changes without breaking critical functionality. CI/CD with proper testing catches errors early and avoids the nightmare of the “white screen of death” on the live site. Code reviews and standards ensure security and quality are maintained consistently, which is especially important if the theme will be maintained for years.

## **Phase 8: Security Hardening and Maintenance**

**Goal:** Implement security best practices in the theme and overall setup, ensuring the site and user data remain safe from common vulnerabilities.

* **Update PHP and WP Compatibility:** Ensure the new theme is fully compatible with **PHP 8.3+** and the latest WordPress core. Refactor any deprecated PHP functions. Namespacing all custom functions and classes in the theme will prevent conflicts with plugins or core. For example, use `LCF_Theme_*` prefixes or PHP namespaces for any custom functions to avoid collisions (legacy code might have had generic names).

* **Secure Configuration:** Add critical security constants in configuration: e.g., **disable file editing** in WP admin (`DISALLOW_FILE_EDIT true`) so no one can inject code via the theme/plugin editors. Unless needed, consider disabling **XML-RPC** which is often exploited (if the site doesn’t use Jetpack or remote posting, you can remove or block `xmlrpc.php`).

* **Sanitization & Escaping:** Go through all custom code (theme or plugin) and apply proper **input sanitization and output escaping** in line with WordPress standards. This means any time user input is processed (e.g., form submissions, query parameters), use functions like `sanitize_text_field()`, `sanitize_email()`, etc., to clean it. And whenever printing data to the front-end (like form values, or data from the database), escape it with `esc_html()`, `esc_attr()`, etc. This prevents XSS and other injection attacks. Although much of a block theme’s structure is in static HTML templates, any dynamic PHP (in `functions.php` or custom blocks) must be reviewed for security.

* **Least Privilege & Roles:** Review user accounts and roles on the site. Ensure administrators are limited to those who truly need that level. For forms that allow file uploads or other capabilities, make sure permission checks are in place. If the site has an online donation handling, security around those forms (like nonces, CAPTCHA or anti-spam, validation) should be in place to prevent misuse.

* **Authentication Hardening:** Enforce **Two-Factor Authentication (2FA)** for admin logins if possible. This might be done via a security plugin or a custom SSO solution if internal. Also implement **login rate limiting** to prevent brute force attacks (many security plugins offer this, or use a simple custom function or server config). While this is more operational than theme code, it’s part of the overall modernization to make the site resilient.

* **HTTPS & Content Security:** Ensure the site runs fully on HTTPS. Add security headers via the server or a plugin (Content Security Policy, XSS Protection, etc.) to harden the front-end. For instance, if we use external resources (CDN scripts, etc.), list them in CSP. The theme itself should load scripts via `https://` and avoid inline scripts/styles where possible (to make CSP easier to implement).

* **Regular Maintenance Plan:** As part of modernization, schedule regular maintenance tasks: update WordPress core, theme, and plugins on a routine (say monthly) after testing in staging. Keep dependencies (like Bootstrap if retained, or Tailwind, etc.) updated to get security patches. This should be documented as a responsibility for someone on the team. Also, consider periodic security scans (there are services or plugins that can scan for malware or vulnerabilities).

* **Justification:** Strengthening security is essential for protecting the Foundation’s reputation and user trust. Non-profits can be targets for defacement or data theft; thus we take a proactive stance. The theme itself should not introduce vulnerabilities – by sanitizing inputs and escaping outputs, we reduce risk of XSS or database injection through theme functionality. Back-end hardening like 2FA and file edit restrictions ensure that even if an admin credential is compromised, the damage is limited. A secure site that is kept updated and audited will safeguard the Foundation’s mission online.

---

By following this phased plan, the Luke Cyr Foundation’s website will transform into a **future-ready, performance-optimized, and accessible platform**. The shift to a block-based theme architecture will empower content creators, while the focus on lean CSS, fast loading, and best practices will ensure every visitor — whether a donor on broadband or a veteran on a slow connection — can access the site quickly and easily. Each step is grounded in modern WordPress standards and justifiable improvements, setting the stage for a site that is not only easier to maintain but also resilient and impactful in fulfilling the Foundation’s mission.

