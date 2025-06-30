# Citation Checker

A Python library for parsing, validating, and checking academic citations.

## Project Overview

The Citation Checker project is designed to parse and validate academic citations from various formats (currently focusing on APA style). It provides tools to extract citations from text, parse them into structured data, and validate their components.

## Project Structure

```
citationchecker/
├── demo_parser.py              # Demo script showing basic usage
├── requirements.txt            # Project dependencies
└── src/                        # Main source code
    ├── __init__.py
    ├── main.py                 # Command-line interface
    ├── models/                 # Data models
    │   ├── __init__.py
    │   └── citation.py         # Citation data model
    ├── parsers/                # Citation parsers
    │   ├── __init__.py
    │   ├── apa_parser.py       # APA citation format parser
    │   ├── base_parser.py      # Abstract base parser class
    │   └── utils.py            # Utility functions for parsing
    └── tests/                  # Test suite
        ├── __init__.py
        ├── test_apa_parser.py  # Tests for APA parser
        ├── test_simple.py      # Simple tests
        └── test_data/          # Test data
            └── apa_citations.json  # Sample APA citations for testing
```

## Components

### 1. Citation Model (`src/models/citation.py`)

The `Citation` class provides a standardized data structure for representing citations across different formats. It includes:

- **Basic Components**: authors, year, title, source
- **Additional Metadata**: DOI, URL, volume, issue, pages, publisher
- **Citation Metadata**: type, format, original text, confidence score

### 2. Parser Architecture

#### Base Parser (`src/parsers/base_parser.py`)

The `BaseCitationParser` is an abstract base class that defines the interface for all citation parsers:

- `parse(text)`: Parse a single citation string
- `extract_citations(text)`: Extract all citations from a larger text
- `calculate_confidence(citation)`: Calculate confidence score for a parsed citation

#### APA Parser (`src/parsers/apa_parser.py`)

The `APACitationParser` implements parsing for APA citation format:

- Uses regex patterns to identify and extract citation components
- Handles various APA citation formats and variations
- Provides fallback parsing for non-standard formats
- Calculates confidence scores based on completeness

#### MLA Parser (`src/parsers/mla_parser.py`)

The `MLACitationParser` implements parsing for MLA citation format (9th edition):

- **Supported MLA Citation Types**:
  - **Journal Articles**: `Smith, John. "Article Title." Journal Name, vol. 15, no. 2, 2020, pp. 45-67.`
  - **Books**: `Author, First. Book Title. Publisher, Year.`
  - **Web Articles**: `Author, First. "Web Article." Website Name, Date, URL.`
  - **Book Chapters**: `Author. "Chapter Title." Book Title, edited by Editor, Publisher, Year, pp. 45-67.`
  - **Multiple Authors**: `Smith, John, and Mary Johnson. "Title." Source, Date.`
  - **Et Al. Citations**: `Smith, John, et al. "Title." Source, Date.`
  - **No Author Citations**: `"Article Title." Source, Date.`

- **Key Features**:
  - Handles MLA-specific author formats (full names, "and", "et al.")
  - Extracts volume/issue in MLA format ("vol. X, no. Y")
  - Processes various MLA date formats (year only, full dates)
  - Identifies citation types (article, book, web, chapter)
  - Supports Works Cited section extraction
  - High accuracy parsing with confidence scoring

#### Utility Functions (`src/parsers/utils.py`)

Helper functions for parsing citations:

- `normalize_author_name`: Standardize author name formats
- `extract_urls`, `extract_year`, `extract_doi`: Extract specific components
- `clean_title`: Remove unnecessary punctuation from titles
- `extract_pages`, `extract_volume_issue`: Extract publication details

### 3. Command-line Interface (`src/main.py`)

Provides command-line functionality:

- Parse citations from files or text input
- Select parser type (currently only APA)
- Output in various formats (text, JSON, CSV)
- Save results to file or display in console

### 4. Demo Script (`demo_parser.py`)

Demonstrates basic usage of the citation parser:

- Parsing a single citation
- Extracting citations from text with multiple citations

## How It Works

### Citation Parsing Process

1. **Input**: The system takes text containing one or more citations.

2. **Extraction**: For text with multiple citations, the `extract_citations` method:
   - Looks for a references section
   - Splits text into potential citation chunks
   - Filters chunks based on length and basic patterns

3. **Parsing**: For each potential citation, the `parse` method:
   - Attempts to match the complete citation pattern
   - If successful, extracts all components (authors, year, title, etc.)
   - If not, falls back to extracting components individually

4. **Author Processing**: The `_process_authors` method:
   - Handles APA-style author lists with "&" separators
   - Correctly pairs lastnames with initials
   - Normalizes author name formats

5. **Metadata Extraction**: Additional methods extract:
   - DOI and URL information
   - Volume and issue numbers
   - Page ranges
   - Citation type (article, book, etc.)

6. **Confidence Calculation**: The system calculates a confidence score (0-1) based on:
   - Presence of required fields (authors, title, source)
   - Presence of additional metadata (year, DOI, volume, etc.)

7. **Output**: Returns structured `Citation` objects with all extracted data

### Example Usage

#### Command Line

```bash
# Parse citations from a file
python -m src.main -f path/to/paper.txt -o json

# Parse a specific citation
python -m src.main -t "Smith, J. (2020). Title. Journal, 15(2), 45-67." -o text
```

#### Python API

**APA Citations:**
```python
from src.parsers.apa_parser import APACitationParser

# Initialize parser
parser = APACitationParser()

# Parse a single citation
citation = parser.parse("Smith, J. (2020). Title. Journal, 15(2), 45-67.")

# Extract citations from text
citations = parser.extract_citations(text_with_multiple_citations)
```

**MLA Citations:**
```python
from src.parsers.mla_parser import MLACitationParser

# Initialize MLA parser
mla_parser = MLACitationParser()

# Parse different MLA citation types
journal_citation = mla_parser.parse('Smith, John. "Digital Libraries in Research." Journal of Information Science, vol. 45, no. 3, 2020, pp. 234-250.')

book_citation = mla_parser.parse('Wilson, Robert. Understanding Academic Citations. University Press, 2019.')

web_citation = mla_parser.parse('Chen, Li. "Citation Tools Review." TechReview Online, 15 Mar. 2023, https://www.techreview.com/citations.')

# Extract from Works Cited section
works_cited_text = """
Works Cited

Smith, John. "Digital Libraries in Research." Journal of Information Science, vol. 45, no. 3, 2020, pp. 234-250.

Johnson, Mary, and David Brown. "Citation Analysis." Academic Writing Quarterly, vol. 12, no. 4, 2021, pp. 45-67.
"""

citations = mla_parser.extract_citations(works_cited_text)
print(f"Found {len(citations)} MLA citations")

# Access citation data
for citation in citations:
    print(f"Authors: {citation.authors}")
    print(f"Title: {citation.title}")
    print(f"Source: {citation.source}")
    print(f"Year: {citation.year}")
    print(f"Confidence: {citation.confidence_score:.2f}")
```

## Testing

The project includes a comprehensive test suite:

- `test_apa_parser.py`: Tests for the APA citation parser
- `test_mla_parser.py`: Tests for the MLA citation parser
- `test_data/apa_citations.json`: Sample APA citations for testing
- `test_data/mla_citations.json`: Sample MLA citations for testing

Run tests with pytest:

```bash
python -m pytest
```

### Test Coverage

**APA Parser Tests:**
- Parametrized tests for various APA citation formats
- Author parsing with multiple authors and "&" separators
- Volume, issue, and page extraction
- DOI and URL extraction
- Confidence score validation

**MLA Parser Tests:**
- Journal articles with volume/issue/pages
- Book citations with publishers
- Web citations with URLs and dates
- Book chapters with editors
- Multiple author formats ("and", "et al.")
- No-author citations
- Works Cited section extraction
- Edge cases and error handling

## Phase 3: Web Crawling Module ✅

The Citation Checker now includes a comprehensive web crawling system for extracting content from academic and general websites to support citation verification.

### Web Crawling Architecture

#### Base Crawler (`src/crawlers/base_crawler.py`)

The `BaseCrawler` is an abstract base class that provides:

- **Session Management**: Configured HTTP sessions with retry strategies
- **Robots.txt Compliance**: Automatic checking of robots.txt files
- **Rate Limiting**: Built-in delays between requests
- **Error Handling**: Comprehensive error handling and logging
- **Content Cleaning**: Text normalization and cleaning utilities

#### Academic Crawler (`src/crawlers/academic_crawler.py`)

The `AcademicCrawler` specializes in extracting content from academic databases:

- **Supported Academic Domains**:
  - **arXiv**: Paper titles, authors, abstracts, subjects, submission dates
  - **PubMed**: Medical papers, authors, abstracts, journals, DOIs
  - **IEEE Xplore**: Technical papers and conference proceedings
  - **ACM Digital Library**: Computer science publications
  - **Nature, Science, PLOS**: Scientific journals
  - **ScienceDirect, Springer**: Academic publishers
  - **JSTOR**: Academic archives
  - **Google Scholar**: Citation data

- **Extraction Features**:
  - Domain-specific content extraction strategies
  - Metadata extraction (authors, titles, abstracts, DOIs)
  - Publication information (journals, dates, volumes)
  - Citation counts and related papers

#### Web Crawler (`src/crawlers/web_crawler.py`)

The `WebCrawler` handles general websites with multiple extraction strategies:

- **Content Extraction Methods**:
  1. **Newspaper3k**: Optimized for news articles and blog posts
  2. **Trafilatura**: General-purpose content extraction
  3. **Heuristic-based**: Fallback method using HTML structure analysis

- **Features**:
  - Automatic method selection based on content type
  - Main content identification and extraction
  - Metadata extraction (titles, descriptions, authors)
  - Clean text output with proper formatting

#### Crawler Manager (`src/crawlers/crawler_manager.py`)

The `CrawlerManager` provides a unified interface for all crawling operations:

- **Automatic Crawler Selection**: Chooses the best crawler for each URL
- **Concurrent Crawling**: Multi-threaded crawling for improved performance
- **Statistics Tracking**: Comprehensive crawling statistics and metrics
- **Error Handling**: Graceful handling of failed requests
- **Resource Management**: Proper cleanup and resource management

### Usage Examples

#### Basic Web Crawling

```python
from src.crawlers.crawler_manager import CrawlerManager

# Initialize crawler manager
with CrawlerManager(max_workers=3, default_timeout=10) as manager:
    # Crawl a single URL
    result = manager.crawl_url("https://arxiv.org/abs/2101.00001")
    
    if result.success:
        print(f"Title: {result.metadata.get('title', 'N/A')}")
        print(f"Content: {result.content[:200]}...")
        print(f"Response Time: {result.response_time:.2f}s")
    else:
        print(f"Error: {result.error}")
```

#### Batch Crawling

```python
# Crawl multiple URLs concurrently
urls = [
    "https://arxiv.org/abs/2101.00001",
    "https://pubmed.ncbi.nlm.nih.gov/12345678",
    "https://example.com/article"
]

with CrawlerManager() as manager:
    results = manager.crawl_urls(urls)
    
    for result in results:
        print(f"URL: {result.url}")
        print(f"Success: {result.success}")
        if result.success:
            print(f"Content Length: {len(result.content)} chars")
```

#### Citation Integration

```python
# Crawl URLs from citation objects
citations = [
    {'title': 'Paper 1', 'url': 'https://arxiv.org/abs/2101.00001'},
    {'title': 'Paper 2', 'url': 'https://example.com/paper'}
]

with CrawlerManager() as manager:
    results = manager.crawl_from_citations(citations)
    
    for i, result in enumerate(results):
        citation = citations[i]
        print(f"Citation: {citation['title']}")
        print(f"Verification: {'✅ Available' if result.success else '❌ Failed'}")
```

#### Crawler Statistics

```python
with CrawlerManager() as manager:
    # Perform crawling operations...
    results = manager.crawl_urls(urls)
    
    # Get detailed statistics
    stats = manager.get_stats()
    print(f"Total Crawls: {stats['total_crawls']}")
    print(f"Success Rate: {stats['success_rate']:.1%}")
    print(f"Average Time: {stats['avg_crawl_time']:.2f}s")
    print(f"Crawler Usage: {stats['crawler_usage']}")
```

### Supported Domains

The web crawling system supports:

**📚 Academic Domains (13 specialized crawlers):**
- arxiv.org - arXiv preprint server
- pubmed.ncbi.nlm.nih.gov - PubMed medical database
- ieeexplore.ieee.org - IEEE Xplore digital library
- dl.acm.org - ACM Digital Library
- nature.com - Nature Publishing Group
- science.org - Science Magazine
- plos.org - PLOS journals
- sciencedirect.com - Elsevier ScienceDirect
- springer.com - Springer publications
- jstor.org - JSTOR academic archives
- scholar.google.com - Google Scholar
- biorxiv.org - bioRxiv preprint server
- medrxiv.org - medRxiv preprint server

**🌐 General Web Support:**
- All HTTP/HTTPS websites with intelligent content extraction
- Automatic fallback strategies for different content types
- Respectful crawling with robots.txt compliance

### Demo Script

Run the web crawler demo to see it in action:

```bash
python demo_crawler.py
```

The demo script demonstrates:
- Crawler selection for different URL types
- Concurrent crawling performance
- Content extraction from various sources
- Integration with citation verification workflow
- Statistics and performance metrics

### Testing

Comprehensive test suite for web crawling functionality:

```bash
# Run crawler tests
python -m pytest src/tests/test_crawlers.py -v
```

**Test Coverage:**
- Individual crawler functionality
- Content extraction methods
- Crawler manager coordination
- Concurrent crawling performance
- Error handling and edge cases
- Integration with citation system

### Rate Limiting and Ethics

The web crawling system is designed to be respectful:

- **Robots.txt Compliance**: Automatically checks and respects robots.txt files
- **Rate Limiting**: Built-in delays between requests (1 second default)
- **Timeout Handling**: Configurable timeouts to avoid hanging requests
- **User Agent**: Identifies itself as an educational research tool
- **Retry Strategy**: Intelligent retry with exponential backoff
- **Resource Cleanup**: Proper session management and resource cleanup

## Future Development

Potential areas for expansion:

1. Support for additional citation formats (Chicago, IEEE, etc.)
2. Enhanced citation verification using crawled content
3. Citation correction suggestions based on web content
4. Web interface for citation checking
5. Batch processing capabilities
6. Integration with more academic databases
7. Advanced content analysis and similarity detection
