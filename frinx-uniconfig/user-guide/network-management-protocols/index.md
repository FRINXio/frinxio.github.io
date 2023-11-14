---
label: Device Installation
order: 9000
---


# Device installation

## Device installation

Guide to installation mechanisms along with CLI, NETCONF and gNMI examples.

## UniConfig CLI

The CLI southbound plugin allows UniConfig to communicate with CLI devices that do not support NETCONF or other programmatic APIs. The CLI service module uses YANG models and implements a translation logic to send and receive structured data to and from CLI devices.

## UniConfig Netconf

NETCONF is an Internet Engineering Task Force (IETF) protocol used to configure and monitor devices in the network. It can create, recover, update and delete configurations of network devices.

NETCONF operations are overlaid on the Remote Procedure Call (RPC) layer and may be described in either XML or JSON.

## UniConfig-native CLI

UniConfig-native CLI allows you to configure CLI-enabled devices using YANG models that describe configuration commands.

In a UniConfig-native CLI deployment, translation units are defined only by YANG models and device-specific characteristics used for parsing and serializing commands. Readers and writers are automatically created and provided to the translation registry (i.e., the user does not need to write them individually).

YANG models can be constructed by following well-defined rules explained in the Developer guide.

## UniConfig gNMI

The gRPC Network Management Interface (gNMI) is used to configure and monitor devices in the network. It can create, update, and delete configurations of network devices as well as susbcriptions to telemetry streams.

gNMI operations are performed via Remote Procedure Call (RPC) developed by Google. All gNMI messages within gRPC service definition are defined as protocol buffers (proto3).

## UniConfig gNOI

The gRPC Network Operations Interface (gNOI) is used to execute operational commands on network devices. The gNOI protocol supports certificate management, bootstrapping and OS installation service.

All gNOI messages within gRPC service definition are defined as protocol buffers (proto3).

Network management protocols are used in the southbound API of the UniConfig Lighty distribution to install and communicate with devices.

The following protocols are currently supported:

* NETCONF (Network Configuration Protocol)
* SSH / TELNET
* gNMI/gNOI (Network Management/Operations interface)
