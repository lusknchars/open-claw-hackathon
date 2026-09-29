# Repro Relay on the Plow OpenClaw base. Pinned by digest: this is base-771198a9
# of plow-pbc/plow-openclaw-agent.
FROM public.ecr.aws/e1h7x4a2/plow-cloud-agents:base-771198a9609dcef54d44843e7da5329c17fa51b4@sha256:f1e7c421b97a80f1bd17015f96daceb965f350a241f7edc7e4d856a0e3a6f8f5
LABEL org.opencontainers.image.title="Repro Relay" \
      org.opencontainers.image.source="https://github.com/lusknchars/repro-relay" \
      org.opencontainers.image.licenses="MIT"
ENV AGENT_ID=repro-relay
# The skill scripts keep their records under HERMES_HOME (the name predates the
# port); point it into the base's persistent state volume.
ENV HERMES_HOME=/var/lib/plow/relay
USER root
# The persona goes after the base prompt, so the base's Plow chat rules stay.
COPY agent/runtime/persona.md /tmp/persona.md
RUN printf '\n' >> /opt/plow/prompt/AGENTS.md \
 && cat /tmp/persona.md >> /opt/plow/prompt/AGENTS.md && rm /tmp/persona.md
COPY agent/skills/ /opt/plow/skills/
COPY agent/LICENSE agent/NOTICE /usr/share/doc/repro-relay/
COPY agent/vendor/ /usr/share/doc/repro-relay/vendor/
RUN chmod -R a+rX /opt/plow/skills
USER node
