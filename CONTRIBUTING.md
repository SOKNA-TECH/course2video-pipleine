# Contributing to Video-to-Course Pipeline

Thank you for your interest in contributing to the Video-to-Course Pipeline! This document provides guidelines and instructions for contributing.

## Authors and Attribution

**Primary Authors:**
- SOKNA RnD Team
- Fatma Meawad

**AI Models Used:**
- Gemini Pro 2.5 (built-in Google Colab for assistance)

## How to Contribute

### Reporting Issues
- Use the GitHub issue tracker to report bugs or suggest features
- Include as much detail as possible: steps to reproduce, expected vs actual behavior
- For bugs, include your environment details (Colab version, Python version, etc.)

### Submitting Changes
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Test your changes in Google Colab
5. Commit your changes (`git commit -m 'Add some amazing feature'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

### Pull Request Guidelines
- Keep PRs focused on a single feature or bug fix
- Update documentation if needed
- Ensure notebooks run correctly in Google Colab
- Follow the existing code style and structure

## Development Environment

This project is designed to run in **Google Colab**. To test changes:

1. Open the notebook in Google Colab
2. Make sure all cells run without errors
3. Test with sample data if available
4. Verify output formats match the expected schemas

## Code Style

- Use descriptive variable names
- Add comments for complex logic
- Follow Python PEP 8 style guidelines
- Keep notebooks clean and well-organized

## Project Structure

```
video-to-course-pipeline/
├── notebooks/          # Jupyter notebooks for each pipeline step
├── config/            # Configuration examples
├── schemas/           # JSON schemas for data formats
├── README.md          # Project documentation
├── CONTRIBUTING.md    # This file
├── CODE_OF_CONDUCT.md # Community guidelines
└── LICENSE            # MIT License
```

## Questions?

Feel free to open an issue for questions about the project or contribution process.