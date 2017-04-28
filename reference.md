Part IV. Reference
==================

Table of Contents
------------------

27. dCache Clients

    The SRM Client Suite

        srmcp — Copy a file from or to an SRM or between two SRMs.
        srmstage — Request staging of a file.

    dccp

        dccp — Copy a file from or to a dCache server.

28. dCache Cell Commands

    Common Cell Commands

        pin — Adds a comment to the pinboard.
        info — Print info about the cell.
        dump pinboard — Dump the full pinboard of the cell to a file.
        show pinboard — Print a part of the pinboard of the cell to STDOUT.

    PnfsManager Commands

        pnfsidof — Print the pnfs id of a file given by its global path.
        flags remove — Remove a flag from a file.
        flags ls — List the flags of a file.
        flags set — Set a flag for a file.
        metadataof — Print the meta-data of a file.
        pathfinder — Print the global or local path of a file from its PNFS id.
        set meta — Set the meta-data of a file.
        storageinfoof — Print the storage info of a file.
        cacheinfoof — Print the cache info of a file.

    Pool Commands

        rep ls — List the files currently in the repository of the pool.
        st set max active — Set the maximum number of active store transfers.
        rh set max active — Set the maximum number of active restore transfers.
        mover set max active — Set the maximum number of active client transfers.
        mover set max active -queue=p2p — Set the maximum number of active pool-to-pool server transfers.
        pp set max active — Set the value used for scaling the performance cost of pool-to-pool client transfers analogous to the other set max active-commands.
        set gap — Set the gap parameter - the size of free space below which it will be assumed that the pool is full within the cost calculations.
        set breakeven — Set the breakeven parameter - used within the cost calculations.
        mover ls — List the active and waiting client transfer requests.
        migration cache — Caches replicas on other pools.
        migration cancel — Cancels a migration job
        migration clear — Removes completed migration jobs.
        migration concurrency — Adjusts the concurrency of a job.
        migration copy — Copies files to other pools.
        migration info — Shows detailed information about a migration job.
        migration ls — Lists all migration jobs.
        migration move — Moves replicas to other pools.
        migration suspend — Suspends a migration job.
        migration resume — Resumes a suspended migration job.

    PoolManager Commands

        rc ls — List the requests currently handled by the PoolManager
        cm ls — List information about the pools in the cost module cache.
        set pool decision — Set the factors for the calculation of the total costs of the pools.

29. dCache Default Port Values
30. Glossary
