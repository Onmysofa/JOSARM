#ifndef JOS_INC_MMU_H
#define JOS_INC_MMU_H

/*
 * This file contains definitions for the x86 memory management unit (MMU),
 * including paging- and segmentation-related data structures and constants,
 * the %cr0, %cr4, and %eflags registers, and traps.
 */

/*
 *
 *	Part 1.  Paging data structures and constants.
 *
 */
	/* The MMU has two translation tables. Table 0 covers the bottom
	 * of the address space, from 0x00000000, and deals with between
	 * 32MB to 4GB of the virtual address space.
	 * Translation table 1 covers the rest of the memory. For now,
	 * both tables are set to the same thing, and, by default, table
	 * 0 manages the entire virtual address space. Later on, the
	 * table 0 register will be pointed to process-specific tables for
	 * each process's virtual memory
	 */

	/* Set up translation table
	 * ARM1176JZF-S manual, 6-39
	 *
	 * The memory is divided in to 4096 1MB sections. Most of these are
	 * unmapped (resulting in prefetch/data aborts), except
	 * 0x8000000-0xa1000000, which are mapped to 0x00000000-0x2a000000
	 * (physical memory and peripherals), and the kernel code and data
	 *
	 * Memory privilege is set by APX/AP bits (3 bits in total)
	 * APX is at bit 15 of the sector definition, while AP are at bits
	 * 10 and 11.
	 *
	 * APX AP            Privileged    Unprivileged
	 *  1  11 (0x8c00) = read-only     read-only
	 *  1  01 (0x8400) = read-only     no access
	 *  0  10 (0x0800) = read-write    read-only
	 *  0  01 (0x0400) = read-write    no-access
	 *
	 * eXecute Never (XN) is at bit 4 (0x10) - sections with this flag
	 * cannot be executes, even by privileged processor modes
	 *
	 * Bits 0 and 1 identify the table entry type
	 * 0 or 3 = translation fault (3 is reserved and shouldn't be used)
	 * 1 = course page table
	 * 2 = section or supersection
	 */
// A linear address 'la' has a three-part structure as follows:
// Ruogu: Modified version of page table, which only has one level!
// +--------12------------+------------20-------------+
// |      Page Table      |      Offset within Page   |
// |         Index        |                           |
// +----------------------+---------------------------+
// \---- PTX(la)--------/ \---- PGOFF(la) -----------/
// \---------- PGNUM(la) ----------/
//
// The PDX, PTX, PGOFF, and PGNUM macros decompose linear addresses as shown.
// To construct a linear address la from PDX(la), PTX(la), and PGOFF(la),
// use PGADDR(PDX(la), PTX(la), PGOFF(la)).

// page number field of address
#define PGNUM(la)	(((uintptr_t) (la)) >> PTXSHIFT)

// page table index
#define PTX(la)		((((uintptr_t) (la)) >> PTXSHIFT) & 0xFFF)

// offset in page
#define PGOFF(la)	(((uintptr_t) (la)) & 0xFFFFF)

// construct linear address from indexes and offset
#define PGADDR(d, t, o)	((void*) (0 | (t) << PTXSHIFT | (o)))

// Page directory and page table constants.

#define NPTENTRIES	4096		// page table entries per page table

#define PGSIZE		0x100000	// bytes mapped by a page
#define PGSHIFT		20		// log2(PGSIZE)

#define PTSIZE		0x400000 // bytes mapped by a page directory entry
#define PTSHIFT		32		// log2(PTSIZE)

#define PTXSHIFT	20		// offset of PTX in a linear address

// Ruogu: PTE(Section Entry)
// +--------31-20---------+-19-16--+15-+--14-12---+-11-10-+9+----8-5---+4+3+2+1+0+
// |      Section base    |  Zeros |APX|  ZEROS   |  AP   | | Domain   |X|C|B| | |
// |        address       |        |   |          |       |0|          |N| | |1|0|
// +----------------------+--------+---+----------+-------+-+----------+-+-+-+-+-+
// Page table/directory entry flags.
#define PTE_AP_X	0x8C00	// AP and APX bits
#define PTE_P_MASK	0x0003
#define PTE_P		0x0002	// Present
#define PTE_NX		0x0010	// Execute never
#define PTE_C		0x0008	// Cacheable
#define PTE_B		0x0004	// Bufferable
#define PTE_SRUR	0x8C00  // Superviser reads only & user reads only
#define PTE_SRO		0x8400  // Superviser reads only
#define PTE_SWO		0x0400	// Superviser writes only
#define PTE_SWUR	0x0800	// Superviser writes & user reads
#define PTE_SWUW	0x8C00	// Superviser writes & user writes


// Address in page table or page directory entry
#define PTE_ADDR(pte)	((physaddr_t) (pte) & ~0xFFFFF)


// Eflags register
#define FL_CF		0x00000001	// Carry Flag
#define FL_PF		0x00000004	// Parity Flag
#define FL_AF		0x00000010	// Auxiliary carry Flag
#define FL_ZF		0x00000040	// Zero Flag
#define FL_SF		0x00000080	// Sign Flag
#define FL_TF		0x00000100	// Trap Flag
#define FL_IF		0x00000200	// Interrupt Flag
#define FL_DF		0x00000400	// Direction Flag
#define FL_OF		0x00000800	// Overflow Flag
#define FL_IOPL_MASK	0x00003000	// I/O Privilege Level bitmask
#define FL_IOPL_0	0x00000000	//   IOPL == 0
#define FL_IOPL_1	0x00001000	//   IOPL == 1
#define FL_IOPL_2	0x00002000	//   IOPL == 2
#define FL_IOPL_3	0x00003000	//   IOPL == 3
#define FL_NT		0x00004000	// Nested Task
#define FL_RF		0x00010000	// Resume Flag
#define FL_VM		0x00020000	// Virtual 8086 mode
#define FL_AC		0x00040000	// Alignment Check
#define FL_VIF		0x00080000	// Virtual Interrupt Flag
#define FL_VIP		0x00100000	// Virtual Interrupt Pending
#define FL_ID		0x00200000	// ID flag

// Page fault error codes
#define FEC_PR		0x1	// Page fault caused by protection violation
#define FEC_WR		0x2	// Page fault caused by a write
#define FEC_U		0x4	// Page fault occured while in user mode



/*
 *
 *	Part 3.  Traps.
 *
 */

#ifndef __ASSEMBLER__
#include <inc/types.h>

// Task state segment format (as described by the Pentium architecture book)
struct Taskstate {
	uint32_t ts_link;	// Old ts selector
	uintptr_t ts_esp0;	// Stack pointers and segment selectors
	uint16_t ts_ss0;	//   after an increase in privilege level
	uint16_t ts_padding1;
	uintptr_t ts_esp1;
	uint16_t ts_ss1;
	uint16_t ts_padding2;
	uintptr_t ts_esp2;
	uint16_t ts_ss2;
	uint16_t ts_padding3;
	physaddr_t ts_cr3;	// Page directory base
	uintptr_t ts_eip;	// Saved state from last task switch
	uint32_t ts_eflags;
	uint32_t ts_eax;	// More saved state (registers)
	uint32_t ts_ecx;
	uint32_t ts_edx;
	uint32_t ts_ebx;
	uintptr_t ts_esp;
	uintptr_t ts_ebp;
	uint32_t ts_esi;
	uint32_t ts_edi;
	uint16_t ts_es;		// Even more saved state (segment selectors)
	uint16_t ts_padding4;
	uint16_t ts_cs;
	uint16_t ts_padding5;
	uint16_t ts_ss;
	uint16_t ts_padding6;
	uint16_t ts_ds;
	uint16_t ts_padding7;
	uint16_t ts_fs;
	uint16_t ts_padding8;
	uint16_t ts_gs;
	uint16_t ts_padding9;
	uint16_t ts_ldt;
	uint16_t ts_padding10;
	uint16_t ts_t;		// Trap on task switch
	uint16_t ts_iomb;	// I/O map base address
};

// Gate descriptors for interrupts and traps
struct Gatedesc {
	unsigned gd_off_15_0 : 16;   // low 16 bits of offset in segment
	unsigned gd_sel : 16;        // segment selector
	unsigned gd_args : 5;        // # args, 0 for interrupt/trap gates
	unsigned gd_rsv1 : 3;        // reserved(should be zero I guess)
	unsigned gd_type : 4;        // type(STS_{TG,IG32,TG32})
	unsigned gd_s : 1;           // must be 0 (system)
	unsigned gd_dpl : 2;         // descriptor(meaning new) privilege level
	unsigned gd_p : 1;           // Present
	unsigned gd_off_31_16 : 16;  // high bits of offset in segment
};

// Set up a normal interrupt/trap gate descriptor.
// - istrap: 1 for a trap (= exception) gate, 0 for an interrupt gate.
    //   see section 9.6.1.3 of the i386 reference: "The difference between
    //   an interrupt gate and a trap gate is in the effect on IF (the
    //   interrupt-enable flag). An interrupt that vectors through an
    //   interrupt gate resets IF, thereby preventing other interrupts from
    //   interfering with the current interrupt handler. A subsequent IRET
    //   instruction restores IF to the value in the EFLAGS image on the
    //   stack. An interrupt through a trap gate does not change IF."
// - sel: Code segment selector for interrupt/trap handler
// - off: Offset in code segment for interrupt/trap handler
// - dpl: Descriptor Privilege Level -
//	  the privilege level required for software to invoke
//	  this interrupt/trap gate explicitly using an int instruction.
#define SETGATE(gate, istrap, sel, off, dpl)			\
{								\
	(gate).gd_off_15_0 = (uint32_t) (off) & 0xffff;		\
	(gate).gd_sel = (sel);					\
	(gate).gd_args = 0;					\
	(gate).gd_rsv1 = 0;					\
	(gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;	\
	(gate).gd_s = 0;					\
	(gate).gd_dpl = (dpl);					\
	(gate).gd_p = 1;					\
	(gate).gd_off_31_16 = (uint32_t) (off) >> 16;		\
}

// Set up a call gate descriptor.
#define SETCALLGATE(gate, sel, off, dpl)           	        \
{								\
	(gate).gd_off_15_0 = (uint32_t) (off) & 0xffff;		\
	(gate).gd_sel = (sel);					\
	(gate).gd_args = 0;					\
	(gate).gd_rsv1 = 0;					\
	(gate).gd_type = STS_CG32;				\
	(gate).gd_s = 0;					\
	(gate).gd_dpl = (dpl);					\
	(gate).gd_p = 1;					\
	(gate).gd_off_31_16 = (uint32_t) (off) >> 16;		\
}

// Pseudo-descriptors used for LGDT, LLDT and LIDT instructions.
struct Pseudodesc {
	uint16_t pd_lim;		// Limit
	uint32_t pd_base;		// Base address
} __attribute__ ((packed));

#endif /* !__ASSEMBLER__ */

#endif /* !JOS_INC_MMU_H */
