# base image
FROM ubuntu:24.04

# install dependency tools
RUN apt-get update && apt-get install -y --no-install-recommends \
    net-tools \
    iptables \
    iproute2 \
    wget \
    curl \
    ca-certificates \
    build-essential \
    cmake \
    git \
    ninja-build \
    pkg-config \
    libssl-dev \
    clang \
    python3 \
    && rm -rf /var/lib/apt/lists/*

# install risc0 tools
RUN curl --proto '=https' --tlsv1.2 --retry 10 --retry-connrefused -fsSL 'https://sh.rustup.rs' | sh -s -- -y
ENV PATH="/root/.cargo/bin:${PATH}"
RUN git clone https://github.com/risc0/risc0.git \
    && cd risc0 && ls && cargo install --path rzup \
    && rzup toolchain build rust \
    && git checkout origin/release-1.1 \
    && cargo install --path risc0/cargo-risczero \
    && rzup install cpp
#RUN curl -L https://risczero.com/install | bash
#RUN rzup install
#RUN rzup install cpp
 
# working directory
WORKDIR /app

# supervisord to manage programs
RUN wget -O supervisord http://public.artifacts.marlin.pro/projects/enclaves/supervisord_master_linux_amd64
RUN chmod +x supervisord

# transparent proxy component inside the enclave to enable outgoing connections
RUN wget -O ip-to-vsock-transparent http://public.artifacts.marlin.pro/projects/enclaves/ip-to-vsock-transparent_v1.0.0_linux_amd64
RUN chmod +x ip-to-vsock-transparent

# key generator to generate static keys
RUN wget -O keygen http://public.artifacts.marlin.pro/projects/enclaves/keygen_v1.0.0_linux_amd64
RUN chmod +x keygen

# attestation server inside the enclave that generates attestations
RUN wget -O attestation-server http://public.artifacts.marlin.pro/projects/enclaves/attestation-server_v1.0.0_linux_amd64
RUN chmod +x attestation-server

# proxy to expose attestation server outside the enclave
RUN wget -O vsock-to-ip http://public.artifacts.marlin.pro/projects/enclaves/vsock-to-ip_v1.0.0_linux_amd64
RUN chmod +x vsock-to-ip

# dnsproxy to provide DNS services inside the enclave
RUN wget -O dnsproxy http://public.artifacts.marlin.pro/projects/enclaves/dnsproxy_v0.46.5_linux_amd64
RUN chmod +x dnsproxy

RUN wget -O oyster-keygen http://public.artifacts.marlin.pro/projects/enclaves/keygen-secp256k1_v1.0.0_linux_amd64
RUN chmod +x oyster-keygen

# supervisord config
COPY supervisord.conf /etc/supervisord.conf

# setup.sh script that will act as entrypoint
COPY setup.sh ./
RUN chmod +x setup.sh

COPY start_prover.sh ./
RUN chmod +x start_prover.sh


COPY kalypso-attestation-prover-cpu ./kalypso-attestation-prover
RUN chmod +x kalypso-attestation-prover

COPY benchmark ./
RUN chmod +x benchmark

COPY .env ./

# entry point
ENTRYPOINT [ "/app/setup.sh" ]
