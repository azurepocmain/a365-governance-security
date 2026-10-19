# Agent 365: Start Here
<link rel="icon" href="articles//a365.svg" type="image/x-icon" >

## Overview

As organizations adopt agents across different platforms, keeping track of their identities, access, and activity gets more complicated. That's where Microsoft Agent 365 comes in.

Agent 365 provides a centralized control plane for managing, securing, and governing agents through the Microsoft 365 admin center. It brings together administration, identity, security, and compliance capabilities for agents built on Microsoft platforms and supported third party platforms.
<img width="1680" height="818" alt="image" src="https://github.com/user-attachments/assets/26cb031f-27e2-4c17-bf5c-0d4835ae93ee" />
<img width="2148" height="735" alt="image" src="https://github.com/user-attachments/assets/8c8efd4a-6eed-46a5-a1a2-b14e31d02dda" />


We'll start with agent identity blueprints and individual agent identities. These provide the identity foundation, while authentication and authorization determine how an agent signs in and what it can access. For custom built agents, we'll look at how to integrate the supported authentication and telemetry components into the agent's code.

Next, we'll use OpenTelemetry (OTel) to capture activity from agent runs and tool calls. We'll cover the Agent 365 telemetry export path and how to send telemetry to Application Insights alongside Purview collection poliies, where supported. Identity and telemetry work together here, but they serve different purposes: identity controls access, while telemetry helps us understand what happened.

We'll also cover MCP servers in your Azure environment and how to use the Agent 365 CLI for supported MCP registration workflows.

From there, we'll explore how Microsoft Purview can collect agent requests and responses where supported, and how to configure Microsoft Defender to monitor activity, detect threats, and block supported malicious actions.

We'll take a closer look at Microsoft Entra, including how agent identity provisioning and registration vary by platform. We'll also explore Microsoft Entra Global Secure Access and the visibility and access controls it can provide for supported agent traffic.

The goal is to bring those signals together so we can follow an agent's activity, investigate its interactions, and spot issues, including for custom built agents hosted outside Microsoft. The level of detail depends on the instrumentation, integrations, and services we've enabled.

## Prerequisites

Before we get started, confirm that your environment has the licensing for the capabilities you plan to use. Microsoft Agent 365 is included with Microsoft 365 E7 and is also available as a standalone subscription for eligible Microsoft 365 plans.

We'll also use Microsoft Purview, Microsoft Defender, and Microsoft Entra throughout this setup. Check the licensing, feature availability, and configuration requirements for the specific capabilities you want to enable in those services.

For the current licensing and feature details, see the [Microsoft Agent 365 service description](https://learn.microsoft.com/en-us/office365/servicedescriptions/microsoft-agent-365/microsoft-agent-365).




***DISCLAIMER: Sample Code is provided for the purpose of illustration only and is not intended to be used in a production environment unless thorough testing has been conducted by the app and database teams. THIS SAMPLE CODE AND ANY RELATED INFORMATION ARE PROVIDED “AS IS” WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESSED OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE IMPLIED WARRANTIES OF MERCHANTABILITY AND/OR FITNESS FOR A PARTICULAR PURPOSE. We grant You a nonexclusive, royalty-free right to use and modify the Sample Code and to reproduce and distribute the object code form of the Sample Code, provided that. You agree: (i) to not use Our name, logo, or trademarks to market Your software product in which the Sample Code is embedded; (ii) to include a valid copyright notice on Your software product in which the Sample Code is embedded; and (iii) to indemnify, hold harmless, and defend Us and Our suppliers from and against any claims or lawsuits, including attorneys fees, that arise or result from the use or distribution or use of the Sample Code.***






