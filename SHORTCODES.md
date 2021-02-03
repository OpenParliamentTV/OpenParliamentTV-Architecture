### Open Parliament TV Shortcodes

This document describes the naming conventions for shortcodes within the *Open Parliament TV* architecture. The **hierarchy** is always:
* Instance (eg. Germany)
	* Parliaments (eg. National Parliament, Regional Parliament X, etc.) 

#
#### Instances 

Open Parliament TV Instances are usually **country specific** or **transnational** and shall be referenced by their [ISO 3166 Alpha-2 (2-letter) code](https://www.iso.org/obp/ui/#search/code/). 

**Examples:**

| Instance| Shortcode | 
| :------------- | :----------: | 
| Germany | `DE` |
| Canada | `CA` | 
| Europe | `EU` | 
| ... | ... | 

These codes **shall be applied to**: 
* Domains (eg. `de.openparliament.tv`)
* Instance-specific paths / filenames (eg. `config.DE.json`)

In **special cases** (eg. where cultural or regional conventions don't match with the *ISO 3166* standardisation), alternative short codes can be used (eg. Catalunya: `CAT`). In this case, the instance shall be handled **analogous to transnational parliaments**.

With the exception of domains and subdomains, these codes shall always be in **UPPERCASE** notation.


#
#### Parliaments

Parliaments are subdivisions of *Open Parliament TV* Instances and shall be referenced by their ISO 3166 Alpha-2 (2-letter) code (for *national* or *transnational* parliaments) or their respective subdivision (eg. [3166:DE](https://www.iso.org/obp/ui/#iso:code:3166:DE))

| Parliament| Shortcode | 
| :------------- | :----------: | 
| German Bundestag | `DE` |
| Abgeordnetenhaus Berlin | `DE-BE` |
| Landtag Bavaria | `DE-BY` |
| ... | ... | 
| European Parliament |`EU`| 

These codes **shall be applied to**: 
* Parliament-specific paths / filenames (eg. `/parsers/DE-BY/`)
* Prefixes (eg. Media ID: `DE-0170004024`)

For parliaments without ISO 3166 subdivision codes, **custom codes** can be defined, based on the instance shortcode. Example: 

| Parliament| Shortcode | 
| :------------- | :----------: | 
| New York City Council | `US-NYC` |

These codes shall always be always be in **UPPERCASE** notation.

#
#### Languages 

For languages, [ISO 639-2 Alpha-2](https://www.loc.gov/standards/iso639-2/php/code_list.php) (2-letter) codes shall be used. In special cases (eg. to avoid overlaps with instance / country codes), the respective Alpha-3 (3-letter) codes can be used as an alternative.

These codes **shall be applied to**:
* Language-specific paths / filenames (`/lang/lang_en.json`)

These codes shall always be always be in **lowercase** notation.

