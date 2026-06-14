# Memahami Ownership

_Ownership_ (Kepemilikan) adalah fitur paling unik di Rust dan punya pengaruh 
yang sangat dalem buat sisa bahasanya. Fitur ini bikin Rust bisa ngasih jaminan 
_memory safety_ tanpa butuh _garbage collector_, jadi penting sekali buat kita 
paham gimana cara kerja _ownership_. Di bab ini, kita bakal bahas soal 
_ownership_ barengan sama beberapa fitur terkait lainnya: _borrowing_, _slices_, 
dan gimana Rust nyusun data di memori.
