# n8n Video Reel Generator

An automated video reel generator that converts text into engaging video content with AI-powered text-to-speech and animated subtitles. Perfect for creating social media reels, motivational quotes, and educational content.

## Features

- **Text-to-Speech Conversion**: Uses Piper TTS with neural voice models for natural-sounding speech
- **Automatic Subtitle Generation**: Creates perfectly timed subtitles (SRT format) with ~10 words per segment
- **Video Rendering**: Generates vertical videos (1080x1920) optimized for social media platforms
- **Webhook Integration**: Easy API-based workflow automation via n8n
- **Customizable**: Supports custom titles, output names, and text content

## Prerequisites

Before running this project, ensure you have the following installed:

- **Python 3.x** (with virtual environment)
- **n8n** (workflow automation tool)
- **FFmpeg** (for video rendering)
- **Piper TTS** (included in the `piper` directory)
- **Windows PowerShell** (for running commands)

## Project Structure

```
n8n-video/
├── piper/                              # Piper TTS engine and voice models
│   ├── piper.exe                       # TTS executable
│   ├── models/                         # Voice model files
│   │   └── en_US-lessac-medium.onnx
│   ├── espeak-ng-data/                 # Phoneme data
│   └── out/                            # TTS output directory
├── reel/                               # Generated video reels and assets
│   ├── *.wav                           # Generated audio files
│   ├── *.srt                           # Generated subtitle files
│   └── *.mp4                           # Final video output
├── workflows/                          # All workflows
│   ├── Self-Help Reel Generator.json   # n8n workflow configuration
├── requirements.txt                    # Python dependencies
├── .gitignore                          # Git ignore rules
└── README.md                      
```

## Installation

### 1. Clone or Download the Project

```bash
git clone <your-repo-url>
cd n8n-video
```

Or download and extract the project to your desired location.

### 2. Create Python Virtual Environment

```powershell
# Create a new virtual environment
python -m venv venv

# Set execution policy (if needed)
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process

# Activate virtual environment
.\venv\Scripts\Activate.ps1
```

### 3. Install Dependencies

Install all required Python packages using the requirements file:

```powershell
pip install -r requirements.txt
```

This will install:
- **moviepy** (2.2.1) - Video editing and rendering
- **numpy** (2.3.4) - Numerical computing
- **pillow** (11.3.0) - Image processing
- **gTTS** (2.5.4) - Google Text-to-Speech (backup option)
- **python-dotenv** (1.1.1) - Environment variable management
- **ffmpeg-python** (0.2.0) - FFmpeg Python bindings
- All supporting dependencies

### 4. Download Piper Voice Models

The Piper TTS voice models are not included in the repository due to their large size (60+ MB). Download them separately:

1. Download the `en_US-lessac-medium` model from [Piper Voices](https://github.com/rhasspy/piper/releases)
2. Place the `.onnx` and `.onnx.json` files in the `piper/models/` directory
3. Or use any other Piper voice model and update the workflow configuration

### 5. Install FFmpeg

Download and install FFmpeg from [ffmpeg.org](https://ffmpeg.org/download.html) and add it to your system PATH, or update the workflow with the full FFmpeg path.

### 6. Import n8n Workflow

1. Open your n8n instance (default: `http://localhost:5678`)
2. Create a new workflow
3. Import the workflow from `Self-Help Reel Generator.json`
4. Update the file paths in the workflow if your project location differs

## Usage

### Using the Webhook API

Once the n8n workflow is activated, you can generate reels by sending POST requests to the webhook endpoint:

```powershell
$body = @{
  text    = "Your only limit is your mind."
  title   = "Self-Help Motivation"
  outName = "mind-reel.mp4"
} | ConvertTo-Json

Invoke-RestMethod -Method Post -Uri "http://localhost:5678/webhook/reel-generate" -ContentType "application/json" -Body $body
```

### API Parameters

| Parameter | Type   | Required | Description                              |
|-----------|--------|----------|------------------------------------------|
| `text`    | string | Yes      | The text content to convert into a reel  |
| `title`   | string | No       | Video title (default: first sentence)    |
| `outName` | string | No       | Output filename (default: slugified title)|

### Example Requests

**Simple Request:**
```powershell
$body = @{
  text = "Success is not final. Failure is not fatal. It is the courage to continue that counts."
} | ConvertTo-Json

Invoke-RestMethod -Method Post -Uri "http://localhost:5678/webhook/reel-generate" -ContentType "application/json" -Body $body
```

**Custom Title and Filename:**
```powershell
$body = @{
  text    = "Believe you can and you're halfway there."
  title   = "Theodore Roosevelt"
  outName = "roosevelt-quote.mp4"
} | ConvertTo-Json

Invoke-RestMethod -Method Post -Uri "http://localhost:5678/webhook/reel-generate" -ContentType "application/json" -Body $body
```

## How It Works

### Workflow Pipeline

1. **Webhook Trigger**: Receives POST request with text content
2. **Text Processing**: 
   - Parses and validates input
   - Chunks text into ~10-word segments
   - Generates SRT subtitle timing (3 seconds per chunk)
3. **File Creation**: Creates output directory and writes SRT subtitle file
4. **Text-to-Speech**: 
   - Uses Piper TTS with `en_US-lessac-medium` voice model
   - Generates high-quality WAV audio file
5. **Video Rendering**:
   - Creates 1080x1920 black background video
   - Overlays audio and animated subtitles
   - Exports as H.264 MP4 video
6. **Response**: Returns success message with output file path

### Subtitle Styling

Subtitles are styled with:
- **Font**: Arial, 48pt, Bold
- **Color**: White text with black outline
- **Position**: Bottom-aligned with 100px margin
- **Timing**: 3 seconds per segment with smooth transitions

## Configuration

### Customizing Voice Models

To use different Piper voice models:

1. Download models from [Piper Voices](https://github.com/rhasspy/piper/releases)
2. Place model files in `piper/models/`
3. Update the `PIPER_MODEL` path in the n8n workflow

### Customizing Video Style

Edit the FFmpeg command in the workflow to change:
- Video dimensions: `-s=1080x1920`
- Background color: `color=c=black`
- Subtitle font: `FontName=Arial`
- Font size: `FontSize=48`
- Text color: `PrimaryColour=&HFFFFFF&`

## Troubleshooting

### Common Issues

**Issue**: "Execution policy restriction"
```powershell
# Solution: Run with bypass
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
```

**Issue**: "FFmpeg not found"
```
Solution: Install FFmpeg and add to system PATH or update the workflow with the full FFmpeg path
```

**Issue**: "Piper TTS fails"
```
Solution: Check that piper.exe and model files are in the correct locations
```

### Logs

- Piper stdout: `piper/piper_stdout.txt`
- Piper stderr: `piper/piper_stderr.txt`

## Examples

See the `reel` directory for example outputs:
- `mind-reel.mp4` - "Your only limit is your mind"
- `self-help-motivation.wav` - Generated audio file
- `self-help-motivation.srt` - Generated subtitle file

## Dependencies

All Python dependencies are listed in `requirements.txt`. Install them with:

```powershell
pip install -r requirements.txt
```

**Key packages:**
- moviepy==2.2.1
- numpy==2.3.4
- pillow==11.3.0
- gTTS==2.5.4
- ffmpeg-python==0.2.0
- python-dotenv==1.1.1

For a complete list, see [requirements.txt](requirements.txt).

## Version Control

This project includes a `.gitignore` file that excludes:
- Generated video files (`reel/*.mp4`, `reel/*.wav`, `reel/*.srt`)
- Virtual environment (`venv/`)
- Python cache files (`__pycache__/`)
- Local configuration files
- IDE-specific files

You can safely commit the project to Git without including generated files or sensitive data.

## License

This project is provided as-is for educational and creative purposes.

## Contributing

Feel free to fork this project and customize it for your needs. Suggestions for improvements:
- Add more voice models
- Support for multiple languages
- Custom video backgrounds and animations
- Batch processing of multiple texts
- Integration with social media APIs for direct posting

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review n8n workflow logs
3. Verify all dependencies are installed correctly
4. Ensure file paths match your system configuration

---

**Enjoy creating video reels!**

