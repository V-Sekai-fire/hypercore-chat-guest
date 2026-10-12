# hypercore-chat-guest

A peer-to-peer chat client on hypercore, built as a godot-sandbox guest ELF
(riscv64, binary-translated by the sandbox). Guest code is Lean 4. The
human-facing layer is GDScript compiled to `.sgd`.

Status: parked. Nothing builds from this repository yet.

## Design

- The guest is a static riscv64 Lean 4 ELF, built the way
  `V-Sekai-fire/interactor-lean-guest` builds one (`lean_guest_executable`).
- The guest is sandboxed. It has no socket or file system calls. All I/O goes
  through the godot-sandbox guest API (`api.hpp`, `syscalls.h`). This
  repository uses those syscalls and does not change them.
- I/O shim: a small C layer that calls Godot classes through the API
  (`ClassDB::instantiate("PacketPeerUDP")`, `FileAccess`, then method calls)
  and exposes `@[extern]` functions to Lean. If the sandbox refuses those
  classes, the fallback is host-mediated I/O: the guest exports
  `ADD_API_FUNCTION` entry points that take and return `PackedByteArray`, and
  the host does the UDP and file work.
- Protocol layers, bottom up: UDX framing, DHT RPC, HyperDHT with Noise
  handshake, hypercore (Merkle tree log), then chat on top (autobase and
  hyperdb schemas).

## References

- The open-source upstream JavaScript packages are the source of truth for
  the wire format: `compact-encoding`, `udx-native`, `dht-rpc`, `hyperdht`,
  `hypercore`, `autobase` and `hyperdb`. Port from them.
- `hyperswarm-peer`: an existing C++ port of udx, dht-rpc and hyperdht. Its
  licence is not declared, so it is read as a reference only. Its JS-generated
  test vectors are the oracles for the Lean codecs.

## Next steps

1. Build a probe guest that links `sandbox_api` next to the Lean runtime.
   Check that `ClassDB::instantiate("PacketPeerUDP")` works inside the sandbox.
2. Write the C shim and its Lean `@[extern]` bindings (UDP send/receive,
   file read/write). Test them with a GDScript host scene.
3. Port the protocol layers in Lean, bottom up, starting with UDX and the
   compact-encoding codec, checked against the upstream JS byte for byte.
4. Add CI that builds the ELF with the riscv64 sysroot.
