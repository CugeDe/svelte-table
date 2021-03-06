<script>
    import SvelteTable from "../src/SvelteTable.svelte";
    // import SvelteTable from "svelte-table";
    import faker from "faker";

    let iconAsc     = "↑";
    let iconDesc    = "↓";
    let sortBy      = "id";
    let sortOrder   = 1;

    $: selectedCols = ["id", "first_name", "last_name", "email", "gender"];

    $: numRows      = 50;
    $: seed         = 5;
    $: data         = [];
    $: cols         = selectedCols.map((key) => Object.assign(COLUMNS[key]));
    $: {
        if (numRows !== NaN && seed !== NaN) {
            generateData();
        }
    }

    generateData();

    function generateData() {
        faker.seed(seed);

        data    = Array(numRows)
            .fill("")
            .map((_n, i) => {
                let d           = {
                    id:             i,
                    email:          '',
                    first_name:     faker.name.firstName(),
                    gender:         faker.random.number(1) ? "Female" : "Male",
                    ip_address:     `192.168.${faker.random.number(128)}.${faker.random.number(255)}`,
                    last_name:      faker.name.lastName(),
                };

                d.email       = `${d.first_name[0].toLowerCase()}.${d.last_name.toLowerCase()}@zipit.org.ca`;

                return d;
            });
    }

    const COLUMNS       = {
        id:                 {
            key:                "id",
            title:              "ID",
            value:              (v) => v.id,
            sortable:           true,
            filterOptions:      (rows) => {
                let nums            = {};

                rows.forEach((row) => {
                    let num             = Math.floor(row.id / 10);

                    if (nums[num] === undefined) {
                        nums[num]       = {
                            name:           `${num * 10} to ${(num + 1) * 10}`,
                            value:          num,
                        };
                    }
                });

                // fix order
                nums = Object.entries(nums).sort().reduce((o, [k, v]) => ((o[k] = v), o), {});

                return Object.values(nums);
            },
            filterValue:        (v) => Math.floor(v.id / 10),
            headerClass:        "text-left",
        },
        first_name:         {
            key:                "first_name",
            title:              "FIRST NAME",
            value:              (v) => v.first_name,
            sortable:           true,
            filterOptions:      (rows) => {
                let letrs           = {};

                rows.forEach((row) => {
                    let letr            = row.first_name.charAt(0);

                    if (letrs[letr] === undefined) {
                        letrs[letr]     = {
                            name:           `${letr.toUpperCase()}`,
                            value:          letr.toLowerCase(),
                        };
                    }
                });

                // fix order
                letrs = Object.entries(letrs).sort().reduce((o, [k, v]) => ((o[k] = v), o), {});

                return Object.values(letrs);
            },
            filterValue:        (v) => v.first_name.charAt(0).toLowerCase(),
        },
        last_name:          {
            key:                "last_name",
            title:              "LAST NAME",
            value:              (v) => v.last_name,
            sortable:           true,
            filterOptions:      (rows) => {
                let letrs           = {};
                rows.forEach((row) => {
                    let letr            = row.last_name.charAt(0);

                    if (letrs[letr] === undefined) {
                        letrs[letr]     = {
                            name:           `${letr.toUpperCase()}`,
                            value:          letr.toLowerCase(),
                        };
                    }
                });

                // fix order
                letrs = Object.entries(letrs).sort().reduce((o, [k, v]) => ((o[k] = v), o), {});

                return Object.values(letrs);
            },
            filterValue:        (v) => v.last_name.charAt(0).toLowerCase(),
        },
        email:              {
            key:                "email",
            title:              "EMAIL",
            value:              (v) => v.email,
            sortable:           true,
            class:              "text-center",
        },
        gender:             {
            key:                "gender",
            title:              "GENDER",
            value:              (v) => v.gender,
            renderValue:        (v) => {
                const classNames    = [`g_${v.gender.toLowerCase()}`];

                let icon            = v.gender.toLowerCase() === "female" ? "&female;" : "";
                icon                = v.gender.toLowerCase() === "male" ? "&male;" : icon;

                return `<span class="${classNames.join(" ")}">${icon} ${v.gender}</span>`;
            },
            sortable:           false,
            filterOptions:      ["Male", "Female"],
        },
        ip_address:         {
            key:                "ip_address",
            title:              "IP ADDRESS",
            value:              (v) => {
                // Compute IP address to a number to provide a correct sorting
                const computedIP = Array.from(v.ip_address.split('.').reverse().entries())
                    .map(([n, v]) => ([n, parseInt(v)]))
                    .reduce(
                        (accumulator, [n, currentValue]) => accumulator + (currentValue * (Math.pow(256, n))),
                        0,
                    );

                return computedIP;
            },
            renderValue:        (v) => v.ip_address,
            sortable:           true,
        },
    };
</script>

<style>
    div :global(.g_female) {
        /* UI Properties */
        color:      #f9e;
    }
    div :global(.g_male) {
        /* UI Properties */
        color:      #99e;
    }
    div :global(.text-center) {
        /* UI Properties */
        text-align: center;
    }
    div :global(.text-left) {
        /* UI Properties */
        text-align: left;
    }
</style>

<div>
    <h1>SvelteTable example 3</h1>
    <p />
    <div class="row">
        <button
            class="waves-effect waves-light btn"
            on:click={() => {
                sortBy = 'id';
            }}
            disabled={sortBy === 'id'}
        >
            SORT BY ID
        </button>

        <button
            class="waves-effect waves-light btn"
            on:click={() => {
                sortBy = 'first_name';
            }}
            disabled={sortBy === 'first_name'}
        >
            SORT BY FIRST NAME
        </button>

        <button
            class="waves-effect waves-light btn"
            on:click={() => {
                sortBy = 'last_name';
            }}
            disabled={sortBy === 'last_name'}
        >
            SORT BY LAST NAME
        </button>

        <button
            class="waves-effect waves-light btn"
            on:click={() => {
                if (selectedCols.length > 5) {
                    selectedCols = ['id', 'first_name', 'last_name', 'email', 'gender'];
                } else {
                    selectedCols = ['id', 'first_name', 'last_name', 'email', 'gender', 'ip_address'];
                }
            }}
        >
            {!selectedCols.includes('ip_address') ? 'Show' : 'Hide'} IP Address
        </button>

        <button
            class="waves-effect waves-light btn"
            on:click={() => {
                sortOrder = 1;
            }}
            disabled={sortOrder === 1}
            style="float: right;"
        >
            SORT {iconAsc}
        </button>

        <button
            class="waves-effect waves-light btn"
            on:click={() => {
                sortOrder = -1;
            }}
            disabled={sortOrder === -1}
            style="float: right;"
        >
            SORT {iconDesc}
        </button>
    </div>

    <div class="row">
        <div class="col s6">
            <label>numRows: {numRows}<input type="range" bind:value={numRows} /></label>
        </div>
        <div class="col s6">
            <label>seed: {seed}<input type="range" bind:value={seed} /></label>
        </div>
    </div>

    <SvelteTable
        columns={cols}
        rows={data}
        bind:sortBy
        bind:sortOrder
        classNameTable={['table highlight responsive-table']}
        classNameThead={['thead']}
        classNameSelect={['custom-select']}
        on:clickCol={(e) => console.log('clickCol:', e)}
        on:clickRow={(e) => console.log('clickRow:', e)}
        on:clickCell={(e) => console.log('clickCell:', e)}
    />
</div>
