```mermaid
graph LR;
classDef className fill:#f9f,stroke:#333,stroke-width:4px;
	subgraph sA[Cluster];
        A[[air-server]];
        B[[air-node-02]];
        C[[air-node-03]];
        D[[air-node-04]];
        D1[[air-node-05]];
        E[(air-storage)];
        G{{2 * A100}};
        H{{8 * A100}};
        I{{6 * A100}};
        J{{8 * A100}};
        J1{{8 * A100}};

        subgraph sB[Slurm System]
            A===>|slurm|B;
            A==>|slurm|C;
            A==>|slurm|D;
            A==>|slurm|D1;
            A=====G;
            B===H;
            C===I;
            D===J;
            D1===J1;
        end
        A----|nfs|E;
        B----|nfs|E;
        C----|nfs|E;
        D----|nfs|E;
        D1----|nfs|E;

    end
    F([103.242.175.247]);

    F-.-A;
    style sA fill:#EED,stroke:#DDD;
    style sB fill:#DEE;
```

