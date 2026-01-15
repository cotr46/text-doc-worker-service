# Text Document Processing Worker Service

Background worker service for processing documents and text analysis jobs.

## Service Information

**Service Name**: text-doc-worker-service-v2  
**Region**: asia-southeast2  
**URL**: https://text-doc-worker-service-v2-lh5pr6ewdq-et.a.run.app  
**Platform**: Google Cloud Run

## Deployment

### Automatic Deployment

Service automatically deploys when code is pushed to `main` branch:

```bash
git add .
git commit -m "Your changes"
git push origin main
```

Deployment takes 5-10 minutes. Monitor progress:
```bash
gcloud builds list --project=bni-prod-dma-bnimove-ai --limit=5
```

### Manual Deployment

```bash
gcloud run deploy text-doc-worker-service-v2 \
  --source . \
  --region=asia-southeast2 \
  --project=bni-prod-dma-bnimove-ai
```

## Architecture

### Processing Flow

```
Pub/Sub â†’ Worker Service â†’ AI Model â†’ Firestore
```

1. Worker subscribes to Pub/Sub topic
2. Receives job messages from API Service
3. Downloads files from Cloud Storage
4. Processes with appropriate AI model
5. Saves results to Firestore
6. Publishes completion message

### Supported Processing Types

**Document Processing**:
- KTP analysis
- NPWP analysis
- SKU analysis
- NIB analysis
- BPKB analysis
- SHM/SHGB analysis

**Text Analysis**:
- PEP (Politically Exposed Person) screening
- Negative news analysis
- Law involvement screening

## Configuration

### Environment Variables

- `GOOGLE_CLOUD_PROJECT` - GCP project ID (default: bni-prod-dma-bnimove-ai)
- `PUBSUB_SUBSCRIPTION` - Pub/Sub subscription name (default: document-processing-worker)
- `PUBSUB_RESULTS_TOPIC` - Results topic (default: document-processing-results)
- `FIRESTORE_DATABASE` - Firestore database (default: document-processing-firestore)
- `OPENWEBUI_API_KEY` - AI model API key
- `OPENWEBUI_BASE_URL` - AI model base URL (default: https://nexus-bnimove-369455734154.asia-southeast2.run.app)
- `MAX_FILE_SIZE_MB` - Maximum file size (default: 100)

### Resources

- Memory: 4Gi
- CPU: 2
- Min instances: 1
- Max instances: 10
- Timeout: 300 seconds (5 minutes)

## Processing Details

### Document Processing

**PDF Processing**:
- Extracts text and images from PDF
- Analyzes with AI model
- Supports multi-page documents
- Handles scanned documents (OCR)

**Image Processing**:
- Supports JPG, JPEG, PNG formats
- Image quality optimization
- Text extraction via OCR

### Text Analysis

**PEP Analysis**:
- Model: politically-exposed-person-v2
- Temperature: 1.0 (required by model)
- Timeout: 300 seconds
- Entity types: person only

**Negative News**:
- Model: negative-news
- Entity types: person, corporate

**Law Involvement**:
- Model: law-involvement
- Entity types: person, corporate

## Development

### Local Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Run worker
python worker.py
```

### Testing

Worker service runs continuously and processes messages from Pub/Sub. To test:

1. Submit job via API Service
2. Monitor worker logs
3. Check Firestore for results

## Monitoring

### View Logs

```bash
gcloud logging read \
  "resource.type=cloud_run_revision AND resource.labels.service_name=text-doc-worker-service-v2" \
  --limit=50 \
  --project=bni-prod-dma-bnimove-ai
```

### View Metrics

Console: https://console.cloud.google.com/run/detail/asia-southeast2/text-doc-worker-service-v2

### Key Metrics

- Messages processed per minute
- Processing time per job
- Success/failure rate
- Model response time
- Memory/CPU utilization

## Troubleshooting

### Common Issues

**Timeout Errors**:
- Increase timeout setting (currently 300s)
- Check AI model response time
- Verify network connectivity

**Model Errors**:
- Verify API key is valid
- Check model availability
- Review temperature settings

**Memory Issues**:
- Monitor memory usage
- Optimize PDF processing
- Reduce concurrent processing

## Support

**Project**: bni-prod-dma-bnimove-ai  
**Region**: asia-southeast2  
**Repository**: https://github.com/cotr46/text-doc-worker-service
