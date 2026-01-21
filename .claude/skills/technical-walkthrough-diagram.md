# Technical Walkthrough Diagram Skill

## Overview
The "Technical Walkthrough Diagram" skill automates the process of fetching documentation, generating visual explanations, and creating accompanying posts. When users request technical diagrams, the tool organizes key information and produces "how-it-works" diagrams plus explanatory content suitable for social media or blogs.

## Core Workflow Steps

1. **Confirm Output Format** - Ask users whether they want:
   - Single tweet (simple format)
   - Twitter thread
   - Blog post (multiple images)

2. **Fetch Content** - Use web tools to retrieve user-specified documentation, GitHub repositories, or URLs. Secondary pages may be fetched if necessary for supplementary information.

3. **Generate Diagrams** - Create visualizations tailored to the chosen format:
   - Twitter: single highlighted diagram
   - Posts: overview diagram plus section-specific sub-diagrams

## Output Structure
Generated materials are stored in: `output/ontshotskills/`

This directory contains both images and written posts.

## Image Generation Guidelines

Prompts must include style descriptions formatted as: *"Generate a hand-drawn style explanatory diagram of technical principles based on the following technical content."* All annotations should use English.

## Post Guidelines

- **Single tweets**: Simple, understandable language without emojis
- **Threads**: Place primary image in first tweet
- **Blog posts**: Design chapter structure first, then generate consistent diagrams per subsection

## Available Tools

**Image Generation**: "pay-per-use-based nano banana text2image"
- API Discovery: GET request to `https://proxy-monetize.fluxapay.xyz/api/b7da8911-ab69-4577-9d90-f9ae6ca34bfa`
- Save base64 responses to files before processing

**Web Fetch**: Custom tool via `ava-agent.fluxapay-qa.xyz/tools/webfetch_magic`

## X402 Payment Protocol

When APIs require payment, use FluxA Wallet endpoints. The process involves:

1. Submit initial request
2. If HTTP 402 received, extract payment requirements
3. Register agent if needed via `https://agentid.fluxapay.xyz/register`
4. Call payment endpoint with AGENT_JWT authorization
5. Retry original request with payment signature header

USDC amounts require 6 decimal places precision.
